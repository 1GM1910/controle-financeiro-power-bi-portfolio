# Automações com Google Apps Script

## Visão Geral

O Google Apps Script funciona como a camada de automação operacional do projeto.

Ele conecta os registros recebidos pela base operacional às regras de negócio necessárias para alimentar o modelo analítico utilizado no Power BI.

## Responsabilidades

Entre as principais responsabilidades da automação estão:

- validar dados recebidos;
- criar registros financeiros;
- gerar parcelas;
- associar compras a cartões e faturas;
- criar valores A Receber para compras de terceiros;
- processar pagamentos de dívidas;
- processar pagamentos de faturas;
- atualizar saldos e status;
- registrar erros e eventos de manutenção;
- permitir reprocessamento de registros.

## Organização por módulos

A solução foi separada em arquivos com responsabilidades específicas:

~~~text
00_Config.gs
01_Utils.gs
02_Logs.gs
03_Validacoes.gs
04_ListasApoio.gs
05_Faturas.gs
06_Dividas.gs
07_Pagamentos.gs
08_Forms.gs
09_Manutencao.gs
10_Triggers.gs
11_Formatacao.gs
~~~

Essa separação melhora legibilidade, manutenção e evolução do código.

## Fluxo simplificado

~~~text
Google Forms
      ↓
Google Sheets
      ↓
Trigger do Apps Script
      ↓
Validação
      ↓
Regra de negócio
      ↓
Atualização das tabelas estruturadas
      ↓
Power BI
~~~

## Exemplo — criação de dívida

Quando uma movimentação exige controle parcelado, a automação:

1. valida os dados;
2. cria o registro da dívida;
3. cria as parcelas;
4. calcula os vencimentos;
5. associa cartão/fatura quando aplicável;
6. atualiza o status inicial.

## Exemplo — pagamento de parcela

Ao registrar um pagamento:

1. a parcela é identificada;
2. o valor pago é registrado;
3. o saldo é recalculado;
4. o status da parcela é atualizado;
5. o status da dívida também pode ser alterado.

Isso permite representar pagamentos totais e parciais.

## Exemplo — pagamento de fatura

O pagamento da fatura é registrado como um evento separado da própria fatura.

A automação atualiza:

- valor pago;
- saldo;
- status da fatura.

## Tratamento de cartão emprestado

Quando o cartão do titular é utilizado para uma compra de terceiro, a solução mantém dois conceitos distintos:

~~~text
Obrigação com o banco
+
Valor a receber do terceiro
~~~

A compra compõe a fatura do cartão e também gera uma dívida do tipo A Receber.

## Validações

A camada de automação ajuda a prevenir inconsistências como:

- campos obrigatórios vazios;
- valores inválidos;
- registros sem referência;
- associação incorreta com cartão ou fatura;
- duplicidade de processamento.

## Logs e manutenção

A versão operacional utiliza estruturas de log para facilitar diagnóstico, manutenção e reprocessamento.

Essas abas e dados operacionais não fazem parte da base pública disponibilizada neste portfólio.

## Segurança

O código público/documentado não deve expor:

- IDs privados de planilhas;
- IDs de formulários;
- URLs internas;
- e-mails;
- tokens;
- chaves;
- credenciais.

Quando necessário, esses valores devem ser substituídos por placeholders.

## Resumo

O Apps Script não é apenas um conector entre Forms e Sheets. Ele representa a camada onde parte relevante das regras de negócio é aplicada antes que os dados cheguem ao modelo analítico.

Essa divisão permite separar claramente:

~~~text
Entrada de dados
↓
Processamento
↓
Armazenamento
↓
Análise
~~~
