# Registro de decisões de arquitetura (ADR)

Cada decisão que molda o projeto tem um ADR curto: contexto, opções, decisão e consequências. Assim o **porquê** não se perde quando o código muda.

## Índice

| ADR | Decisão | Status |
|---|---|---|
| [0001](0001-escopo-v1.md) | Escopo fechado da versão 1 | Aceito |
| [0002](0002-imposto-fora-do-emissor.md) | O emissor não calcula imposto, só confere | Aceito |
| [0003](0003-arquitetura-hexagonal.md) | Arquitetura hexagonal com motor único | Aceito |
| [0004](0004-modelar-por-autorizador.md) | Modelar por autorizador, não por estado | Aceito |
| [0005](0005-patterns-e-ciclo-de-vida-enum.md) | Patterns do núcleo e ciclo de vida como enum | Aceito |
| [0006](0006-envio-assincrono-idempotencia-modelo-hibrido.md) | Envio assíncrono, idempotência pela chave e modelo híbrido | Aceito |
| [0007](0007-numeracao-atomica-concorrencia.md) | Numeração com reserva atômica e concorrência | Aceito |
| [0008](0008-certificado-por-cnpj-pasta-local.md) | Certificado por CNPJ em pasta local | Aceito |
| [0009](0009-idioma-codigo-e-documentacao.md) | Idioma do código e da documentação | Aceito |
| [0010](0010-stack-tecnologica.md) | Stack: Spring Boot 3, Java 21, Maven, PostgreSQL, RabbitMQ | Aceito |

## Quando escrever um ADR

- Escolha entre alternativas reais que afeta mais de um módulo.
- Mudança de uma regra inegociável ou de um item do escopo.
- Adoção ou troca de tecnologia, biblioteca central ou pattern.

Não precisa de ADR: detalhe de implementação local, nome de variável, refatoração sem mudança de comportamento.

## Como escrever

1. Copie [`0000-template.md`](0000-template.md) para `NNNN-titulo-em-kebab-case.md`, com o próximo número livre.
2. Mantenha curto: cabe em uma tela.
3. Abra o PR com status **Proposto**. Ao aprovar, mude para **Aceito** e atualize o índice acima.
4. ADR aceito não é editado no mérito. Para mudar a decisão, crie um novo ADR e marque o antigo como **Substituído por**.
5. Itens ainda não confirmados ficam marcados como **"a confirmar"** no próprio texto.
