# ADR-0007: Numeração com reserva atômica e concorrência

- **Status:** Aceito
- **Data:** 2026-10-21

## Contexto

O número da nota é sequencial e não pode repetir. Com vários workers e várias instâncias, duas notas podem pedir número ao mesmo tempo. Buracos na sequência são permitidos, mas precisam ser inutilizados junto à SEFAZ. A SEFAZ **não exige** que as notas cheguem em ordem numérica.

## Opções consideradas

1. **`SELECT MAX(numero) + 1`.** Condição de corrida garantida sob concorrência.
2. **Sequence do banco.** Atômica, mas uma sequence por CNPJ + série é difícil de gerenciar e não permite reaproveitar número.
3. **Tabela de contadores com reserva atômica.** Um `UPDATE ... RETURNING` (ou lock de linha) por pedido.
4. **Contador em memória ou cache distribuído.** Perde números em queda e cria outra fonte de verdade.

## Decisão

Opção 3, no PostgreSQL.

- O contador é por **CNPJ + modelo + série + ambiente**. Modelo e ambiente entram na chave porque NF-e e NFC-e, e homologação e produção, têm numerações independentes.
- A reserva é uma única instrução atômica:

  ```sql
  UPDATE numeracao
     SET proximo_numero = proximo_numero + 1
   WHERE cnpj = :cnpj AND modelo = :modelo AND serie = :serie AND ambiente = :ambiente
  RETURNING proximo_numero - 1 AS numero;
  ```

- Workers concorrentes **podem enviar fora de ordem**. Não há fila serial por série.
- **Nota rejeitada reusa o mesmo número**: a correção é reenviada com o mesmo número.
- **Nota denegada consome o número**: ele não volta para uso nem é inutilizado.
- **Número abandonado vai para inutilização.** Um job periódico encontra números reservados e não autorizados há mais tempo que o limite configurado e pede a inutilização.
- Antes de inutilizar, o job **consulta pela chave de acesso** ([ADR-0006](0006-envio-assincrono-idempotencia-modelo-hibrido.md)). Número de nota que pode ter sido autorizada nunca é inutilizado às cegas.

## Consequências

- Não há buraco por condição de corrida; buracos só nascem de abandono, e o job os trata.
- A linha do contador vira ponto de contenção por CNPJ + série. Para o volume esperado na v1 é aceitável; se não for, a solução é distribuir notas em mais séries.
- **A confirmar:** o prazo legal para inutilizar números não utilizados. Fica em `INUTILIZACAO_DEADLINE_DAYS`, vazio até a confirmação, e o job não roda sem esse valor.
- **A confirmar:** o critério exato de "abandonado" (tempo desde a reserva sem envio, rejeição sem correção etc.).
