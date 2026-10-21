# ADR-0008: Certificado por CNPJ em pasta local

- **Status:** Aceito
- **Data:** 2026-10-21

## Contexto

Cada nota é assinada (XMLDSig) e enviada por TLS mútuo com o certificado ICP-Brasil do emitente. Um emissor genérico atende vários CNPJs, incluindo filiais que usam o certificado da matriz. O certificado é a credencial mais sensível do sistema: vazamento permite emitir notas em nome da empresa. Cofres (Vault, AWS) estão fora da v1 ([ADR-0001](0001-escopo-v1.md)).

## Opções consideradas

1. **Certificado no banco.** Centraliza, mas mistura segredo com dado de negócio e amplia a superfície de vazamento.
2. **Cofre de segredos.** Ideal em produção, mas fora da v1.
3. **Pasta local organizada por CNPJ, atrás de uma porta.** Simples de operar agora; o cofre entra depois como outro adaptador.

## Decisão

Opção 3.

- **Estrutura:** `.certificados/<cnpj>/arquivo.pfx`. O nome da subpasta é o CNPJ só com caracteres válidos (dígitos e letras maiúsculas), sem pontuação. CNPJ alfanumérico é aceito: nenhum código assume que CNPJ é só número.
- **Senha:** no `.env`, no formato `CNPJ_<cnpj>=senha`.
- **Escolha do certificado:** pelo **CNPJ do emitente** da nota. Se não houver pasta para o CNPJ exato, procura pela **raiz** (8 primeiros caracteres), para que a filial use o certificado da matriz.
- **Autorização antes do certificado:** antes de buscar o certificado, o emissor verifica se a **credencial de quem chamou** pode emitir por aquele CNPJ. Sem permissão, o certificado nem é carregado.
- **Só em memória:** o `.pfx` é lido para memória e nunca é copiado, extraído nem gravado em disco fora da pasta.
- **Validade monitorada:** a data de vencimento é lida no carregamento. Certificado vencido é recusado; a proximidade do vencimento gera alerta (`CERTIFICADO_EXPIRY_ALERT_DAYS`).
- **Porta:** tudo fica atrás de `CertificateProvider`. Na v1 só existe o adaptador de pasta local.
- `.certificados/` e `.env` estão no `.gitignore`, com aviso no README.

## Consequências

- Trocar a origem do certificado é escrever outro adaptador de `CertificateProvider`.
- Quem opera o serviço é responsável por proteger a pasta (permissões, volume criptografado, backup).
- Testes geram certificados descartáveis em tempo de execução. Nenhum certificado, nem de teste, é versionado.
- **A confirmar:** o mecanismo da credencial de quem chama (proposta: chave de API associada a uma lista de CNPJs ou raízes permitidos).
- **A confirmar:** o comportamento quando a pasta tem mais de um `.pfx` (proposta: usar o válido de maior validade e registrar aviso, para facilitar a renovação).
