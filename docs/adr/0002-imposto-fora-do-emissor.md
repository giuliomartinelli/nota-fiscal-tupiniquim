# ADR-0002: O emissor não calcula imposto, só confere

- **Status:** Aceito
- **Data:** 2026-10-21

## Contexto

Calcular imposto depende de regime tributário, NCM, CFOP, benefícios estaduais, substituição tributária e, a partir da reforma tributária, das regras de IBS e CBS. Essas regras mudam com frequência e são do domínio do ERP ou de um motor fiscal especializado. Por outro lado, a SEFAZ rejeita notas cujos totais não fecham, e cada rejeição custa uma ida e volta.

## Opções consideradas

1. **Calcular os impostos no emissor.** Conveniente para quem chama, mas transforma o emissor num motor tributário, com responsabilidade e manutenção enormes.
2. **Repassar os valores sem olhar.** Simples, mas erros de soma e arredondamento só aparecem como rejeição da SEFAZ.
3. **Receber valores calculados e conferir a matemática.** O emissor não decide alíquota nem base; só verifica se os números enviados são coerentes entre si.

## Decisão

Opção 3. O emissor **não calcula imposto**. Os valores chegam calculados de quem chama. Antes de enviar, o último elo da cadeia de validação ([ADR-0005](0005-patterns-e-ciclo-de-vida-enum.md)) confere:

- soma dos itens = totais da nota;
- base × alíquota = valor do imposto, por item;
- arredondamento conforme as regras do layout (casas decimais e tolerância).

Divergência gera erro de validação com o campo exato, e a nota não é enviada.

## Consequências

- O emissor continua genérico e estável quando a legislação tributária muda.
- Quem chama é responsável pela correção tributária. Isso fica explícito no README.
- Conferir não é calcular: o validador nunca preenche nem corrige um valor, só aponta a divergência.
- **A confirmar:** a tolerância de arredondamento aceita pela SEFAZ por campo, com a nota técnica correspondente.
