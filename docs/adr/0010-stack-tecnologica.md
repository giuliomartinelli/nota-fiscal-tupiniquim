# ADR-0010: Stack tecnológica

- **Status:** Aceito
- **Data:** 2026-10-21

## Contexto

As decisões anteriores exigem: Circuit Breaker ([ADR-0005](0005-patterns-e-ciclo-de-vida-enum.md)), fila com atraso e retry ([ADR-0006](0006-envio-assincrono-idempotencia-modelo-hibrido.md)), reserva atômica no banco ([ADR-0007](0007-numeracao-atomica-concorrencia.md)), SOAP com TLS mútuo e assinatura XMLDSig. O projeto também é portfólio, então a stack deve ser reconhecível no mercado.

## Opções consideradas

| Eixo | Opções | Escolha |
|---|---|---|
| Framework | Spring Boot 3, Quarkus, Java puro | **Spring Boot 3** |
| Linguagem e build | Java 21 ou 25; Maven ou Gradle | **Java 21 + Maven** |
| Banco | PostgreSQL, MySQL | **PostgreSQL** |
| Fila | RabbitMQ, Kafka, fila em tabela | **RabbitMQ** |

## Decisão

- **Spring Boot 3:** integração madura com Resilience4j e RabbitMQ, e o mais reconhecido em vagas. Fica só nos adaptadores; o núcleo não depende dele ([ADR-0003](0003-arquitetura-hexagonal.md)).
- **Java 21 (LTS):** virtual threads ajudam na espera curta do modelo híbrido sem custo de thread por requisição.
- **Maven:** o mais comum em projetos fiscais brasileiros e com geração de classes a partir de XSD bem estabelecida.
- **PostgreSQL:** `UPDATE ... RETURNING` e `SELECT ... FOR UPDATE` nativos para a numeração.
- **RabbitMQ:** filas de trabalho com confirmação, fila de mensagens mortas e atraso por mensagem, que encaixam em Command e no backoff de consulta. Kafka é otimizado para log de eventos e replay, não para fila de comandos com retry individual.

## Consequências

- Ambiente local precisa de PostgreSQL e RabbitMQ (via Docker Compose a partir da etapa de código).
- Trocar Spring por outro framework afeta só os adaptadores.
- **A confirmar na etapa de código:** biblioteca de cliente SOAP e de assinatura XMLDSig (JDK nativo ou Apache Santuario).
