# ADR-0004: Modelar por autorizador, não por estado

- **Status:** Aceito
- **Data:** 2026-10-21

## Contexto

Quem autoriza uma nota não é sempre a SEFAZ da UF do emitente. Vários estados usam autorizadores virtuais (SVRS, SVAN), e o mesmo estado pode usar um autorizador para NF-e e outro para NFC-e. URLs, versões de serviço e particularidades pertencem ao autorizador, não ao estado. Modelar por estado repetiria a mesma integração em dezenas de classes.

## Opções consideradas

1. **Uma classe por UF.** Intuitivo, mas duplica SVRS em muitos estados e espalha correções.
2. **Configuração pura (só URLs em arquivo).** Resolve endereços, mas não acomoda particularidades de comportamento.
3. **Uma Strategy por autorizador + Registry que mapeia UF → autorizador.**

## Decisão

Opção 3.

- `Autorizador` é a interface da Strategy. Cada implementação conhece URLs por `Servico` e `Ambiente`, versões e particularidades.
- `AutorizadorRegistry` resolve o autorizador a partir da **UF e do modelo** da nota. O modelo entra na chave porque o mapeamento de NF-e e de NFC-e pode ser diferente para a mesma UF.
- O mapeamento UF + modelo → autorizador fica em **configuração**, transcrito da tabela oficial do Portal Nacional da NF-e. Não se codifica de memória.
- O mesmo Registry resolve o autorizador de contingência (SVC-AN ou SVC-RS) de cada UF, quando a contingência for implementada.
- Na v1 existem duas implementações: SEFAZ SP e SVRS ([ADR-0001](0001-escopo-v1.md)).

## Consequências

- Atender um novo estado servido por SVRS é só uma linha de configuração.
- Atender um autorizador novo é uma implementação + testes com XMLs reais (receita no [CONTRIBUTING](../../CONTRIBUTING.md)).
- O mapeamento precisa ser revisto quando o portal nacional publicar mudanças. A mudança cita a fonte no PR.
- Refinamento em relação ao briefing original: a chave do Registry é UF **+ modelo**, não só UF.
