# ADR-0003: Arquitetura hexagonal com motor único

- **Status:** Aceito
- **Data:** 2026-10-21

## Contexto

O emissor recebe pedidos por REST e por fila na v1, e por gRPC e XML depois ([ADR-0001](0001-escopo-v1.md)). Também depende de recursos externos que vão mudar: origem do certificado, banco, webhook, SEFAZ. Se a regra fiscal ficar espalhada pelos controllers ou pelos listeners, cada nova entrada duplica o fluxo.

## Opções consideradas

1. **Camadas tradicionais (controller → service → repository).** Familiar, mas a fila tende a ganhar seu próprio service e o fluxo se duplica.
2. **Hexagonal (portas e adaptadores).** O núcleo define portas; REST, fila, banco e certificado são adaptadores trocáveis.
3. **Microsserviços separados por operação.** Isolamento alto, mas complexidade desproporcional para a v1.

## Decisão

Opção 2.

- **Portas de entrada:** REST e fila traduzem o pedido para um Command ([ADR-0005](0005-patterns-e-ciclo-de-vida-enum.md)) e chamam o mesmo motor. Nenhuma regra fiscal vive no adaptador de entrada.
- **Núcleo:** fluxo de envio, validação, numeração, ciclo de vida, montagem e assinatura do XML.
- **Portas de saída:** `CertificateProvider`, repositórios, publicação de webhook, cliente SOAP do autorizador.

O núcleo não depende de Spring, RabbitMQ nem JDBC. A dependência aponta sempre de fora para dentro.

## Consequências

- Nova entrada (gRPC, XML) é só um adaptador novo.
- O núcleo é testável sem rede, banco ou fila.
- Exige disciplina: código de framework no núcleo é rejeitado em revisão. Uma regra de arquitetura automatizada (ex.: ArchUnit) deve vigiar isso a partir da etapa de código.
