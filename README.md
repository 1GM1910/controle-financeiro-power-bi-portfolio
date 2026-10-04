# Controle Financeiro Pessoal - Power BI

Projeto de Business Intelligence desenvolvido com **Power BI, DAX, Power Query, Google Sheets, Google Apps Script e Modelagem Dimensional / Star Schema**.

A solução foi criada para centralizar e analisar receitas, despesas, cartões, faturas, dívidas, valores a receber, vencimentos, pagamentos e fluxo de caixa.

> Esta versão de portfólio utiliza somente dados fictícios e sanitizados.

---

## Visão do Dashboard

![Dashboard Home](assets/dashboard/01_home.webp)

O relatório possui oito páginas: Home, Gastos, Empréstimos, Dívidas, Cartões e Faturas, Vencimentos, Pagamentos e Sobre o Projeto.

---

## Problema de Negócio

O principal desafio foi representar corretamente eventos financeiros relacionados, mas com significados distintos: gasto, dívida, parcela, fatura e pagamento.

Entre os cenários tratados estão compras no cartão, cartão emprestado, valores A Pagar e A Receber, dívidas parceladas, pagamentos parciais, faturas em diferentes status, prevenção de dupla contagem e fluxo de caixa.

---

## Arquitetura

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

- **Google Forms:** entrada padronizada das movimentações.
- **Google Sheets:** base operacional.
- **Google Apps Script:** automação, validação e regras de negócio.
- **Power BI:** relacionamentos, DAX, indicadores e visualizações.

---

## Modelo de Dados

O projeto utiliza Modelagem Dimensional.

### Dimensões

~~~text
DIM_Calendario
DIM_Pessoas
DIM_Cartoes
DIM_Categorias
DIM_Contas
DIM_FormasPagamento
~~~

### Fatos

~~~text
FAT_Gastos
FAT_Entradas
FAT_Dividas
FAT_Parcelas_Dividas
FAT_Faturas_Cartao
FAT_Pagamentos_Dividas
FAT_Pagamentos_Faturas
~~~

Cada fato possui granularidade própria. Exemplo:

~~~text
FAT_Dividas            → 1 linha = 1 dívida
FAT_Parcelas_Dividas   → 1 linha = 1 parcela
FAT_Pagamentos_Dividas → 1 linha = 1 pagamento
~~~

---

## Principais Regras de Negócio

### A Pagar
Representa obrigações financeiras do titular.

### A Receber
Representa valores que terceiros devem ao titular.

### Cartão Emprestado

~~~text
Compra para terceiro
      ↓
Valor entra na fatura
      ↓
Dívida A Receber é criada
      ↓
Pagamento do terceiro
      ↓
Saldo a receber é reduzido
~~~

O valor faz parte da fatura do cartão, mas não deve ser interpretado como gasto próprio do titular.

### Prevenção de Dupla Contagem
Parcelas já incorporadas a uma fatura não são somadas novamente em indicadores de vencimento ou fluxo de caixa.

---

## Fluxo de Caixa

Uma compra no cartão é uma despesa, mas a saída de caixa ocorre no pagamento da fatura.

~~~text
Entradas Caixa = Receitas + Recebimentos de valores a receber
~~~

~~~text
Saídas Caixa = Gastos fora do cartão + Pagamentos de faturas + Pagamentos de dívidas fora da fatura
~~~

---

## DAX e Power Query

Entre os conceitos aplicados estão CALCULATE, FILTER, COALESCE, DIVIDE, USERELATIONSHIP, ALL, ALLSELECTED, REMOVEFILTERS, SELECTEDVALUE, variáveis, funções de data, contexto de filtro e parâmetros no Power Query.

A fonte do portfólio foi centralizada em um parâmetro do Power Query, facilitando manutenção e troca de ambiente.

---

## Dados do Portfólio

A base analítica pública está disponível em:

[Controle_Financeiro_Base_Portfolio.xlsx](data/Controle_Financeiro_Base_Portfolio.xlsx)

O arquivo contém somente dimensões e fatos relevantes para demonstração. Respostas brutas de formulários, logs, tabelas auxiliares operacionais e informações pessoais reais foram removidos.

---

## Arquivo Power BI

O relatório completo está disponível no arquivo **Controle_Financeiro_Portfolio.pbix** e pode ser aberto no Power BI Desktop.

---

## Documentação Técnica

- [Visão Geral](docs/01_VISAO_GERAL.md)
- [Arquitetura](docs/02_ARQUITETURA.md)
- [Modelo de Dados](docs/03_MODELO_DE_DADOS.md)
- [Regras de Negócio](docs/04_REGRAS_DE_NEGOCIO.md)
- [Principais Medidas DAX](docs/05_MEDIDAS_DAX.md)
- [Automações com Apps Script](docs/06_AUTOMACOES_APPS_SCRIPT.md)
- [Dashboard](docs/07_DASHBOARD.md)
- [Privacidade dos Dados](docs/DATA_PRIVACY.md)
- [Status do Projeto](docs/PROJECT_STATUS.md)

---

## Tecnologias

Power BI • DAX • Power Query • Google Forms • Google Sheets • Google Apps Script • Modelagem Dimensional • Star Schema • Git • GitHub

---

## Diferenciais

- integração entre coleta, automação e análise;
- modelagem dimensional;
- tratamento de cartão emprestado;
- separação entre A Pagar e A Receber;
- pagamentos parciais;
- decomposição de faturas;
- prevenção de dupla contagem;
- múltiplos contextos de data;
- fluxo de caixa por movimentação efetiva;
- agenda de vencimentos;
- documentação técnica.

---

## Autor

Desenvolvido por **Guilherme Mendonça**.

Projeto criado para aplicação prática, aprendizado e composição de portfólio profissional na área de **Dados e Business Intelligence**.