# Glossário

Lista **fechada** dos termos que ficam em português no código. A decisão está no [ADR-0009](adr/0009-idioma-codigo-e-documentacao.md).

Se um termo fiscal não está aqui, ele ainda não pode ser usado: abra um PR que o adicione a este arquivo antes (ou junto) do código que o usa.

## Regras de escrita

1. **Código em inglês, substantivo fiscal em português.** O sufixo ou verbo técnico fica em inglês: `InutilizacaoService`, `ChaveAcessoGenerator`, `findByChaveAcesso()`, `reserveNumero()`.
2. **Sem acento nem cedilha em identificador.** `Inutilizacao`, `Situacao`, `Emissao`. Na documentação, o termo é escrito com acento normalmente.
3. **Formas por contexto:**

   | Contexto | Forma | Exemplo |
   |---|---|---|
   | Classe, record, interface | PascalCase | `ChaveAcesso` |
   | Variável, método, campo JSON | camelCase | `chaveAcesso` |
   | Constante e valor de enum | UPPER_SNAKE | `EM_PROCESSAMENTO` |
   | Tabela e coluna no banco | snake_case | `chave_acesso` |
   | Variável de ambiente | UPPER_SNAKE | `CERTIFICADOS_DIR` |

4. **Tag do layout oficial não se traduz nem se renomeia.** `chNFe`, `cNF`, `cStat` aparecem como estão quando o código lida diretamente com o XML. Fora do XML, use o termo do glossário (`chaveAcesso`, não `chNFe`).
5. **Uma coisa, um nome.** A coluna "Não usar" lista sinônimos proibidos, em inglês ou em português.

## Termos fiscais (ficam em português)

### Documento e identificação

| No código | Significado | Não usar |
|---|---|---|
| `NotaFiscal` | Documento fiscal eletrônico (NF-e ou NFC-e) tratado pelo emissor. | `Invoice`, `Nfe` como nome genérico, `Documento` |
| `Modelo` | Modelo do documento: `NFE` (55) ou `NFCE` (65). | `DocumentType`, `Model` |
| `ChaveAcesso` | Chave de 44 posições que identifica a nota. Também é a chave de idempotência do envio. Tag `chNFe`. | `AccessKey`, `Chave`, `NfeKey` |
| `CodigoNumerico` | Código aleatório de 8 posições dentro da chave. Nunca igual ao `Numero`. Tag `cNF`. | `RandomCode`, `NumericCode` |
| `DigitoVerificador` | Último dígito da chave, módulo 11. Tag `cDV`. | `CheckDigit`, `Dv` |
| `Numero` | Número sequencial da nota dentro de CNPJ + modelo + série + ambiente. Tag `nNF`. | `Number`, `InvoiceNumber`, `Sequencial` |
| `Serie` | Série da numeração. | `Series` |
| `Emitente` | Quem emite a nota. Define o certificado usado. | `Issuer`, `Sender`, `Company` |
| `Destinatario` | Quem recebe a nota. | `Recipient`, `Customer` |
| `Cnpj` | Cadastro de pessoa jurídica. **Alfanumérico**: nunca tratar como número. | `TaxId`, `long cnpj` |
| `RaizCnpj` | Os 8 primeiros caracteres do CNPJ. Identifica a empresa (matriz e filiais). | `CnpjRoot`, `CnpjBase` |
| `Cpf` | Cadastro de pessoa física. | `TaxId` |
| `Uf` | Unidade federativa. | `State` |
| `Csc` | Código de segurança do contribuinte, usado no QR Code da NFC-e. | `SecurityCode`, `Token` |

### Autorizadores e comunicação

| No código | Significado | Não usar |
|---|---|---|
| `Sefaz` | Secretaria da Fazenda. Usado para o retorno e para o ambiente remoto em geral. | `TaxAuthority`, `Server` |
| `Autorizador` | Quem autoriza a nota para uma UF e modelo: SEFAZ própria (ex.: SP) ou virtual (SVRS, SVAN). É a Strategy da arquitetura. | `Authorizer`, `Provider`, `Estado` |
| `AutorizadorRegistry` | Mapeamento UF + modelo → `Autorizador`. | `StateRegistry`, `AuthorizerMap` |
| `Svrs` / `Svan` | SEFAZ Virtual do Rio Grande do Sul / do Ambiente Nacional. | — |
| `Servico` | Serviço SOAP da SEFAZ: autorização, retorno de autorização, consulta de protocolo, status do serviço, inutilização, recepção de evento. | `WebServiceType`, `Operation` |
| `StatusServico` | Consulta que diz se o autorizador está em operação. Alimenta o Circuit Breaker. | `HealthCheck`, `Ping` |
| `Lote` | Envelope de envio de uma ou mais notas. | `Batch` |
| `Recibo` | Número devolvido quando o lote é processado de forma assíncrona. Usado para consultar o resultado. | `Receipt`, `Ticket` |
| `Protocolo` | Protocolo de autorização ou de evento. **Sem protocolo de autorização a nota não vale.** | `Protocol`, `AuthorizationCode` |
| `RetornoSefaz` | Resultado normalizado pelo Adapter: `AUTORIZADA`, `REJEITADA`, `DENEGADA`, `EM_PROCESSAMENTO`. | `SefazResponse`, `Result` |
| `cStat` / `xMotivo` | Código e texto de retorno da SEFAZ, mantidos com o nome da tag. | `statusCode`, `message` |
| `Ambiente` | `HOMOLOGACAO` ou `PRODUCAO`. Tag `tpAmb`. | `Environment`, `Env`, `Sandbox` |
| `TipoEmissao` | Normal ou uma das contingências. Tag `tpEmis`. | `EmissionType` |
| `Contingencia` | Emissão fora do autorizador normal, quando ele está indisponível. | `Fallback` como termo fiscal, `Contingency` |
| `Svc` | SEFAZ Virtual de Contingência (SVC-AN, SVC-RS). | — |
| `Epec` | Evento Prévio de Emissão em Contingência. | — |

### Operações e eventos

| No código | Significado | Não usar |
|---|---|---|
| `Emissao` | Operação de emitir uma nota. | `Issuance`, `Emission` |
| `Evento` | Evento fiscal no modelo nacional, vinculado a uma chave de acesso. | `Event` para evento fiscal |
| `Cancelamento` | Evento que cancela uma nota autorizada. | `Cancellation`, `Void` |
| `CartaCorrecao` | Evento de carta de correção (CC-e). Não muda a situação da nota. | `CorrectionLetter`, `Cce` isolado |
| `Inutilizacao` | Pedido para inutilizar uma faixa de números nunca autorizados. | `Voiding`, `NumberDisable`, `Cancelamento` |
| `NotaTecnica` | Nota técnica que origina uma regra fiscal. Citar sempre como `NT AAAA.NNN vX.YY`. | `TechNote`, `Spec` |

### Ciclo de vida da nota

Enum `SituacaoNota` (não usar `Status`, `State` nem `NoteStatus`). Os valores abaixo são a base; a lista definitiva sai na etapa do domínio e qualquer valor novo entra aqui primeiro.

| Valor | Significado |
|---|---|
| `RECEBIDA` | Pedido aceito, número reservado, chave gerada. |
| `EM_PROCESSAMENTO` | Enviada, aguardando resultado final da SEFAZ. |
| `AUTORIZADA` | Tem protocolo de autorização. Única situação em que a nota vale. |
| `REJEITADA` | Recusada (pelo emissor ou pela SEFAZ). Pode ser corrigida e reenviada com o mesmo número. |
| `DENEGADA` | Uso denegado pela SEFAZ. O número fica consumido. |
| `CANCELADA` | Autorizada e depois cancelada por evento. |

## Termos que ficam em inglês

Termos técnicos sem equivalente fiscal. Use em inglês, nunca traduza:

`Service`, `Repository`, `Provider`, `Registry`, `Command`, `Handler`, `Validator`, `Adapter`, `Strategy`, `Port`, `Worker`, `Job`, `Webhook`, `CircuitBreaker`, `Retry`, `Backoff`, `Timeout`, `Signer` (assinatura XMLDSig), `Schema` (XSD), `Certificate` (em `CertificateProvider`), `Credential` (credencial de quem chama a API).

> `Certificate` fica em inglês porque o certificado digital é um conceito técnico (ICP-Brasil, X.509), não um termo do layout fiscal. Na documentação, escreva "certificado".

## Como alterar este glossário

- Termo novo ou mudança de forma só por PR que altera este arquivo.
- O PR explica o termo e a coluna "Não usar".
- Renomear um termo existente exige atualizar todo o código no mesmo PR.
