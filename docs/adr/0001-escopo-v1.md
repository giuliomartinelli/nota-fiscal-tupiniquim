# ADR-0001: Escopo fechado da versão 1

- **Status:** Aceito
- **Data:** 2026-10-21

## Contexto

Um emissor fiscal completo cobre vários documentos (NF-e, NFC-e, CT-e, MDF-e, NFS-e), 27 UFs atendidas por autorizadores diferentes, várias formas de entrada e de guarda de certificado. Tentar tudo de uma vez adia a primeira nota autorizada e dificulta provar que a arquitetura funciona.

## Opções consideradas

1. **Cobrir o máximo desde o início.** Atrai mais usuários, mas a v1 nunca fica pronta e cada autorizador traz particularidades.
2. **Uma UF só.** Rápido, mas não prova que o modelo por autorizador ([ADR-0004](0004-modelar-por-autorizador.md)) se sustenta.
3. **Escopo mínimo que prova o modelo.** Dois documentos parecidos, duas entradas, dois autorizadores de naturezas diferentes.

## Decisão

Opção 3. A versão 1 entrega:

- **Documentos:** NF-e (modelo 55) e NFC-e (modelo 65).
- **Entrada:** REST e fila (RabbitMQ), ambas em JSON.
- **Autorizadores:** SEFAZ SP (autorizador próprio) e SVRS (autorizador virtual que atende vários estados). Os dois juntos provam o Registry e a Strategy.
- **Certificado:** A1 (`.pfx`) em pasta local ([ADR-0008](0008-certificado-por-cnpj-pasta-local.md)).
- **Operações:** emissão, cancelamento, carta de correção, inutilização, consulta de situação e de status do serviço.

**Fora da v1**, com ponto de extensão pronto (interface ou porta, sem implementação):

| Fora da v1 | Ponto de extensão |
|---|---|
| gRPC | Nova porta de entrada chamando os mesmos Commands |
| Entrada por XML pronto | Nova porta de entrada que pula a montagem do XML |
| CT-e, MDF-e, NFS-e e outros | Novo `Modelo` com seus validadores e Strategy |
| Demais autorizadores | Nova implementação de `Autorizador` + registro no Registry |
| Cofre de certificados (Vault, AWS) | Novo adaptador de `CertificateProvider` |
| Interface gráfica | Consome a API REST, fora deste repositório |
| Contingência SVC/EPEC efetiva | Fallback do Circuit Breaker ([ADR-0005](0005-patterns-e-ciclo-de-vida-enum.md)) |

## Consequências

- A v1 é pequena o bastante para ser terminada e grande o bastante para validar a arquitetura.
- Pedidos de outros documentos ou UFs na v1 são recusados com referência a este ADR.
- **A confirmar:** onde ficam o CSC e o identificador do CSC da NFC-e (necessários ao QR Code) por emitente. Provável porta própria, análoga a `CertificateProvider`.
- **A confirmar:** a contingência usada pela NFC-e difere da NF-e e deve ser levantada por autorizador antes de implementar o fallback.
