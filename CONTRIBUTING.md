# Como contribuir

Obrigado pelo interesse. Este guia explica onde cada coisa mora, o que nunca pode ser quebrado e como propor mudanças.

Antes de começar, leia o [glossário](docs/glossario.md) (nomes) e os [ADRs](docs/adr/) (o porquê das decisões).

## Mapa da arquitetura

O projeto tem três regiões. A pergunta "onde coloco isso?" quase sempre se resolve pela coluna "O que varia".

| Região | O que mora aqui | O que varia | O que nunca varia |
|---|---|---|---|
| **Núcleo fixo** | Fluxo de envio, validação, numeração, ciclo de vida, montagem e assinatura do XML | Nada por autorizador | As regras da tabela abaixo |
| **Strategy de autorizador** | Uma implementação de `Autorizador` por autorizador (SP, SVRS…) + Adapter de retorno | URLs, versões de serviço, particularidades do envelope e do retorno | O contrato da interface e os quatro resultados de `RetornoSefaz` |
| **Portas e adaptadores** | Entrada: REST, fila. Saída: `CertificateProvider`, repositórios, webhook | Protocolo de entrada, origem do certificado, banco | O núcleo não conhece nenhum adaptador |

Decisões de fundo: [ADR-0003](docs/adr/0003-arquitetura-hexagonal.md) (hexagonal), [ADR-0004](docs/adr/0004-modelar-por-autorizador.md) (por autorizador), [ADR-0005](docs/adr/0005-patterns-e-ciclo-de-vida-enum.md) (patterns).

### Regras do núcleo (valem para todo autorizador)

Nenhuma Strategy pode desligar, pular ou reimplementar estas regras:

- Validação pelo **XSD oficial** antes de enviar.
- Assinatura **XMLDSig** com certificado ICP-Brasil.
- **SOAP sobre TLS mútuo** com o certificado do emitente.
- **Chave de acesso** de 44 posições com dígito verificador módulo 11. O código numérico **nunca** é igual ao número da nota.
- **CNPJ alfanumérico.** Nenhum código assume que CNPJ é só número: nada de `long`, `Integer.parseInt` ou regex só de dígitos.
- **Ambiente travado:** homologação por padrão; produção só com dupla confirmação.
- **Data de emissão com fuso horário.**
- **XML autorizado + protocolo** guardados juntos. Retenção de 5 anos é responsabilidade documentada de quem opera.
- **Eventos** (cancelamento, carta de correção) no modelo nacional.
- **Consulta de status do serviço** alimenta o Circuit Breaker.

## Como adicionar um autorizador

1. **Confirme que é um autorizador novo.** Se a UF é atendida por um autorizador que já existe (ex.: SVRS), basta acrescentar a linha UF + modelo no mapeamento de configuração, citando a tabela do Portal Nacional da NF-e no PR.
2. **Implemente a interface `Autorizador`.** Informe URLs por `Servico` e `Ambiente`, versões de cada serviço e as particularidades do envelope SOAP. Não reimplemente nada do núcleo.
3. **Implemente o Adapter de retorno** se o autorizador tiver alguma particularidade no XML de resposta. O Adapter traduz tudo para `RetornoSefaz`: `AUTORIZADA`, `REJEITADA`, `DENEGADA` ou `EM_PROCESSAMENTO`.
4. **Registre no `AutorizadorRegistry`** as UFs e modelos que ele atende.
5. **Inclua XMLs de retorno reais como teste**, anonimizados, em `src/test/resources/autorizadores/<nome>/`. No mínimo:
   - autorizada;
   - rejeitada;
   - denegada;
   - lote em processamento (recibo);
   - duplicidade;
   - status do serviço em operação e paralisado.
6. **Rode o teste de contrato em homologação.** Nunca em produção.
7. **Atualize a documentação:** tabela de escopo no README (se mudar a v1) e, se o autorizador exigir decisão nova, um ADR.

Anonimizar significa trocar CNPJ, CPF, nomes, endereços, chave de acesso e protocolo por valores fictícios coerentes (chave com DV válido), mantendo a estrutura e os códigos de retorno.

## Regras inegociáveis

Um PR que quebra qualquer uma destas regras não é aceito, independente do resto.

1. **Não calcular imposto.** O emissor confere valores, nunca os calcula nem corrige. Ver [ADR-0002](docs/adr/0002-imposto-fora-do-emissor.md).
2. **Nunca reenviar no escuro.** Timeout ou erro de rede leva a consulta pela chave de acesso. Duplicidade leva a recuperação do protocolo. Ver [ADR-0006](docs/adr/0006-envio-assincrono-idempotencia-modelo-hibrido.md).
3. **Nunca commitar certificado, senha ou `.env`.** Nem de teste: certificados de teste são gerados em tempo de execução. Se acontecer por engano, avise imediatamente; reescrever o histórico não basta, o certificado precisa ser revogado.
4. **Testes só em homologação.** Nenhum teste, script ou exemplo aponta para produção.
5. **Sem protocolo de autorização, a nota não vale.** Nada informa "autorizada" sem protocolo persistido.

## Toda mudança de regra fiscal cita a nota técnica

Regra fiscal (validação, campo novo, mudança de layout, código de rejeição) sempre tem origem num documento oficial. Cite-o:

- **No código**, num comentário curto junto da regra:

  ```java
  // NT 2024.001 v1.10, regra de validação X99
  ```

- **No PR**, no campo "Regra fiscal" do template.

Formato da citação: `NT AAAA.NNN vX.YY`, seguido do item ou regra de validação. Para o manual, use `MOC vX.YY, seção N`. Sem fonte, a regra não entra.

## Commits

Usamos [Conventional Commits](https://www.conventionalcommits.org/pt-br/v1.0.0/):

```text
<tipo>(<escopo>): <descrição no imperativo, em português>
```

**Tipos:** `feat`, `fix`, `docs`, `refactor`, `test`, `build`, `ci`, `chore`.

**Escopos sugeridos:**

| Escopo | Área |
|---|---|
| `nucleo` | Fluxo, validação, assinatura, ciclo de vida |
| `numeracao` | Reserva de número e inutilização |
| `autorizador-sp`, `autorizador-svrs` | Strategy de cada autorizador |
| `rest`, `fila` | Portas de entrada |
| `certificado` | `CertificateProvider` e adaptadores |
| `docs`, `adr` | Documentação e decisões |

Exemplos:

```text
feat(autorizador-svrs): adiciona consulta de status do serviço
fix(numeracao): evita inutilizar número com nota em processamento
docs(adr): registra decisão sobre CSC da NFC-e
```

Mudança que quebra compatibilidade da API usa `!` após o escopo e explica no rodapé `BREAKING CHANGE:`.

## Pull requests

- Use o [template de PR](.github/pull_request_template.md); ele já traz o checklist das regras acima.
- Um assunto por PR.
- Decisão de arquitetura nova ou alterada vem com ADR. Veja [quando escrever um ADR](docs/adr/README.md#quando-escrever-um-adr).
- Termo fiscal novo entra primeiro no [glossário](docs/glossario.md).
