# Privacidade e Dados

Este projeto separa o ambiente financeiro pessoal da versão destinada ao portfólio.

## Ambiente pessoal

O ambiente original utiliza dados financeiros reais e privados.

Esses dados não devem ser enviados ao GitHub.

Arquivos privados devem permanecer armazenados localmente no diretório:

`private/`

Esse diretório é protegido pelo `.gitignore`.

## Ambiente de portfólio

A versão disponível neste repositório utiliza dados fictícios e sanitizados.

A estrutura do projeto foi preservada para representar de forma realista:

- receitas e despesas;
- cartões e faturas;
- dívidas a pagar;
- valores a receber;
- parcelas;
- vencimentos;
- pagamentos;
- fluxo de caixa;
- regras de negócio e indicadores do dashboard.

Nenhuma informação financeira pessoal real deve ser publicada nesta versão.

## Código e integrações

Antes de publicar códigos de automação ou arquivos de configuração, é necessário revisar e remover informações sensíveis, incluindo:

- IDs de planilhas;
- IDs de formulários;
- URLs privadas;
- e-mails pessoais;
- tokens;
- chaves de API;
- credenciais;
- identificadores internos;
- logs com informações pessoais ou operacionais.

Quando necessário, esses valores devem ser substituídos por exemplos ou placeholders.

## Arquivos que não devem ser versionados

Nunca versionar:

- dados financeiros reais;
- arquivos privados;
- credenciais;
- tokens;
- segredos de API;
- arquivos de configuração com informações sensíveis;
- respostas brutas de formulários com dados pessoais;
- logs contendo informações privadas;
- cópias de segurança do ambiente real.

## Regra de publicação

Antes de tornar o repositório público, deve ser realizada uma auditoria final para confirmar que:

1. o PBIX contém apenas dados fictícios;
2. a documentação não expõe informações pessoais;
3. códigos de automação foram sanitizados;
4. IDs, URLs e credenciais privadas foram removidos;
5. somente arquivos adequados para portfólio permanecem versionados.
