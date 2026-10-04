# Arquitetura da Solução

## Visão Geral

O projeto Controle Financeiro Pessoal foi estruturado em camadas para separar responsabilidades e facilitar organização, manutenção e análise dos dados.

~~~text
Google Forms
      ↓
Google Sheets
      ↓
Google Apps Script
      ↓
Modelo Dimensional / Star Schema
      ↓
Power BI
~~~

## 1. Entrada de Dados — Google Forms

O Google Forms padroniza o registro das movimentações, como entradas, gastos, dívidas, valores a receber e pagamentos.

## 2. Base Operacional — Google Sheets

O Google Sheets armazena as tabelas estruturadas utilizadas pela automação e pelo Power BI.

Principais dimensões:

~~~text
DIM_Pessoas
DIM_Cartoes
DIM_Categorias
DIM_Contas
DIM_FormasPagamento
DIM_Calendario
~~~

Principais fatos:

~~~text
FAT_Gastos
FAT_Entradas
FAT_Dividas
FAT_Parcelas_Dividas
FAT_Faturas_Cartao
FAT_Pagamentos_Dividas
FAT_Pagamentos_Faturas
~~~

## 3. Automação — Google Apps Script

O Google Apps Script funciona como camada de processamento e regras de negócio.

Principais responsabilidades:

- validar dados;
- criar registros;
- gerar parcelas;
- associar cartões e faturas;
- processar pagamentos;
- atualizar status;
- registrar logs;
- permitir manutenção e reprocessamento.

A automação foi organizada em módulos:

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

## 4. Modelagem Dimensional — Star Schema

A camada analítica separa entidades de contexto em dimensões e eventos financeiros em tabelas fato.

Exemplo simplificado:

~~~text
              DIM_Pessoas
                   |
DIM_Categorias - FAT_Gastos - DIM_FormasPagamento
                   |
              DIM_Calendario
~~~

## 5. Camada Analítica — Power BI

O Power BI é responsável por relacionamentos, medidas DAX, análise temporal, indicadores, filtros e visualizações.

As páginas são:

- Home;
- Gastos;
- Empréstimos;
- Dívidas;
- Cartões e Faturas;
- Vencimentos;
- Pagamentos;
- Sobre.

## Separação de Responsabilidades

A solução foi desenhada para separar operacional e analítico.

### Operacional

Forms, Sheets e Apps Script recebem, validam e processam os registros.

### Analítico

Power BI e DAX calculam indicadores e apresentam os dados.

## Fluxo de um Registro

~~~text
Usuário registra movimentação
        ↓
Google Forms
        ↓
Google Sheets
        ↓
Google Apps Script
        ↓
Validação e regras de negócio
        ↓
Tabelas estruturadas
        ↓
Power BI
        ↓
Indicadores
~~~

## Exemplo — Cartão Emprestado

~~~text
Compra para terceiro
      ↓
Compra entra na fatura
      ↓
Dívida A Receber criada
      ↓
Parcelas criadas
      ↓
Pagamento recebido
      ↓
Saldo a receber atualizado
~~~

A obrigação com o banco e o valor devido pelo terceiro permanecem separados.

## Datas e DIM_Calendario

O modelo possui diferentes eventos temporais, como DataGasto, DataEntrada, DataVencimento e DataPagamento. Em determinadas medidas, o DAX ativa relações temporais específicas por meio de USERELATIONSHIP.

## Segurança e Privacidade

A versão de portfólio utiliza dados fictícios e sanitizados. IDs privados, URLs internas, e-mails, tokens, chaves, credenciais e dados financeiros reais não devem ser publicados.

## Limitações e Evolução

A arquitetura atual é adequada ao escopo pessoal e de portfólio, porém Google Sheets e Apps Script possuem limites de escala e concorrência.

Uma evolução possível seria:

~~~text
Aplicação
   ↓
API
   ↓
Banco de Dados
   ↓
Pipeline ETL
   ↓
Data Warehouse
   ↓
Power BI
~~~

## Resumo

A arquitetura separa entrada, processamento, armazenamento, modelagem e análise, aplicando conceitos de Dados e Business Intelligence em uma solução integrada.
