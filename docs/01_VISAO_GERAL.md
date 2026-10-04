# Visão Geral do Projeto

## Controle Financeiro Pessoal - Power BI

O projeto Controle Financeiro Pessoal foi desenvolvido com o objetivo de centralizar, organizar e analisar informações financeiras em um único ambiente analítico.

A solução integra coleta de dados, armazenamento, automação, modelagem dimensional e visualização de dados, utilizando uma arquitetura composta por Google Forms, Google Sheets, Google Apps Script e Power BI.

O projeto nasceu de uma necessidade prática de acompanhar diferentes tipos de movimentações financeiras que, quando controladas separadamente, dificultavam a análise da situação financeira como um todo.

Entre essas movimentações estão:

- receitas;
- despesas;
- compras realizadas por diferentes formas de pagamento;
- cartões de crédito;
- faturas;
- compras realizadas para terceiros;
- valores a receber;
- dívidas pessoais;
- parcelamentos;
- vencimentos;
- pagamentos;
- fluxo de caixa.

---

## Problema

Um controle financeiro simples baseado apenas em entradas e despesas não era suficiente para representar todos os cenários necessários.

Existiam situações como:

- compras realizadas no cartão de crédito que somente impactam o caixa quando a fatura é paga;
- compras realizadas para terceiros utilizando um cartão próprio;
- valores que precisam ser recebidos de outras pessoas;
- dívidas pessoais parceladas;
- parcelas vinculadas a faturas;
- pagamentos parciais;
- diferentes datas de fechamento e vencimento de cartões;
- necessidade de acompanhar compromissos futuros;
- risco de contabilizar duas vezes o mesmo compromisso financeiro.

Por esse motivo, foi necessário estruturar regras de negócio específicas e criar um modelo de dados capaz de representar corretamente cada situação.

---

## Solução

A solução foi dividida em quatro camadas principais:

### 1. Coleta de dados

O Google Forms é utilizado como interface de entrada de informações.

Ele permite registrar movimentações financeiras de forma simples e padronizada.

### 2. Armazenamento e processamento

O Google Sheets funciona como base operacional do projeto.

As informações recebidas são organizadas em tabelas que posteriormente alimentam o modelo analítico.

O Google Apps Script é utilizado para automatizar processos, validar informações e aplicar regras de negócio.

### 3. Modelagem de dados

Os dados são estruturados utilizando Modelagem Dimensional em formato Star Schema.

O modelo possui tabelas dimensão responsáveis por armazenar contextos de análise e tabelas fato responsáveis por armazenar os eventos financeiros.

Entre as principais dimensões estão:

- Calendário;
- Pessoas;
- Cartões;
- Contas;
- Categorias;
- Formas de Pagamento.

Entre as principais tabelas fato estão:

- Gastos;
- Entradas;
- Dívidas;
- Parcelas de Dívidas;
- Faturas de Cartão;
- Pagamentos de Dívidas;
- Pagamentos de Faturas.

Essa separação permite criar relacionamentos mais consistentes e facilitar a construção das análises no Power BI.

### 4. Análise e visualização

O Power BI é responsável pela camada analítica do projeto.

Foram desenvolvidas medidas DAX, indicadores e páginas específicas para diferentes áreas do controle financeiro.

O dashboard permite analisar:

- fluxo financeiro;
- receitas;
- despesas;
- gastos por categoria;
- cartões;
- faturas;
- valores a receber;
- dívidas;
- parcelas;
- vencimentos;
- pagamentos;
- compromissos futuros.

---

## Fluxo da solução

```text
Google Forms
      ↓
Google Sheets
      ↓
Google Apps Script
      ↓
Modelo Dimensional
      ↓
Power BI
      ↓
Dashboard Financeiro
