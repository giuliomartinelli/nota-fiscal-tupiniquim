# ADR-0009: Idioma do código e da documentação

- **Status:** Aceito
- **Data:** 2026-10-21

## Contexto

O domínio é brasileiro: os termos vêm da legislação e dos manuais da SEFAZ, sem tradução oficial. O código em Java, por outro lado, convive com bibliotecas e convenções em inglês. Traduzir termos fiscais gera ambiguidade ("AccessKey" ou "InvoiceKey"?) e afasta o código da documentação oficial. Misturar sem regra gera `AccessKey` num arquivo e `chaveAcesso` no outro.

## Opções consideradas

1. **Tudo em inglês.** Padrão internacional, mas traduções inventadas para termos sem equivalente.
2. **Tudo em português.** Fiel ao domínio, mas briga com o vocabulário técnico das bibliotecas e do próprio Java.
3. **Código em inglês com termos fiscais em português, a partir de uma lista fechada.**

## Decisão

Opção 3.

- **Documentação inteira em português.**
- **Código em inglês**, mas **termos fiscais em português**: `ChaveAcesso`, `InutilizacaoService`, `Autorizador`.
- A lista de termos que ficam em português é **fechada** e mora no [glossário](../glossario.md), com as formas por contexto (classe, variável, enum, banco, variável de ambiente) e os sinônimos proibidos.
- Identificadores sem acento nem cedilha.

## Consequências

- O código fala a mesma língua do manual da SEFAZ e do contador.
- O glossário vira dependência de revisão: termo fiscal fora da lista é bloqueado no PR.
- Contribuidores estrangeiros precisam do glossário para entender o domínio, o que é inevitável neste domínio de qualquer forma.
