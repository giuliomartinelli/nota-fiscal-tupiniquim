## O que muda

<!-- Uma ou duas frases. O título do PR segue Conventional Commits, ex.: feat(autorizador-svrs): ... -->

## Por quê

<!-- Problema ou necessidade. Link para issue, se houver. -->

## Tipo

- [ ] `feat`: nova funcionalidade
- [ ] `fix`: correção
- [ ] `docs`: só documentação
- [ ] `refactor`: sem mudança de comportamento
- [ ] `test`: só testes
- [ ] `chore` / `build` / `ci`

## Regra fiscal

- [ ] Este PR **não** altera regra fiscal.
- [ ] Este PR altera regra fiscal e cita a nota técnica de origem:
  <!-- ex.: NT 2024.001 v1.10, regra de validação X99 -->

## Checklist

- [ ] Testes rodaram **só em homologação**. Nenhum teste toca produção.
- [ ] Nenhum certificado, senha ou `.env` no diff.
- [ ] Nenhum cálculo de imposto foi introduzido (só conferência de valores).
- [ ] Nenhum reenvio às cegas: timeout leva a consulta pela chave de acesso.
- [ ] Nomes seguem o [glossário](../docs/glossario.md).
- [ ] Decisão de arquitetura nova ou alterada tem ADR em [`docs/adr/`](../docs/adr/).
- [ ] Autorizador novo ou alterado: XMLs de retorno reais (anonimizados) incluídos como teste.

## Como testar

<!-- Passos para o revisor reproduzir. -->
