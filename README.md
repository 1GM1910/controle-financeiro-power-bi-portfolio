# Controle Financeiro Pessoal - Power BI

Dashboard financeiro desenvolvido com **Power BI, DAX, Power Query, Google Sheets e Google Apps Script**, utilizando **Modelagem Dimensional / Star Schema** para controle de receitas, despesas, cartões, faturas, dívidas, recebimentos, vencimentos e fluxo de caixa.

O projeto foi desenvolvido a partir de uma necessidade prática de centralizar diferentes tipos de movimentações financeiras em uma única solução analítica.

---

## Dashboard

![Dashboard Home](assets/dashboard/01_home.webp)

A solução possui oito páginas analíticas:

- Home
- Gastos
- Empréstimos
- Dívidas
- Cartões e Faturas
- Vencimentos
- Pagamentos
- Sobre o Projeto

---

## Problema de Negócio

O controle financeiro pode se tornar complexo quando diferentes eventos precisam ser acompanhados simultaneamente.

Entre os cenários tratados neste projeto estão:

- receitas;
- despesas;
- compras realizadas no cartão;
- múltiplos cartões;
- faturas;
- dívidas pessoais;
- compras realizadas para terceiros;
- valores a receber;
- parcelas;
- pagamentos parciais;
- vencimentos;
- fluxo de caixa.

O principal desafio foi representar corretamente a relação entre:

```text
Gasto
Dívida
Parcela
Fatura
Pagamento
```

Esses eventos podem estar relacionados, mas possuem significados financeiros diferentes.

---

## Arquitetura da Solução

```text
Google Forms
      ↓
Google Sheets
      ↓
Google Apps Script
      ↓
Modelo Dimensional / Star Schema
      ↓
Power BI
```

### Google Forms

Responsável pela entrada padronizada das movimentações.

### Google Sheets

Utilizado como base operacional para armazenamento dos dados.

### Google Apps Script

Responsável pela automação, validações e aplicação das regras de negócio.

### Modelo Dimensional

Organiza os dados em tabelas dimensão e tabelas fato.

### Power BI

Responsável pela camada analítica, indicadores e visualizações.

---

## Modelo de Dados

O projeto utiliza **Modelagem Dimensional em formato Star Schema**.

As tabelas são separadas entre dimensões e fatos.

### Principais Dimensões

```text
DIM_Calendario
DIM_Pessoas
DIM_Cartoes
DIM_Contas
DIM_Categorias
DIM_FormasPagamento
```

### Principais Tabelas Fato

```text
FAT_Gastos
FAT_Entradas
FAT_Dividas
FAT_Parcelas_Dividas
FAT_Faturas_Cartao
FAT_Pagamentos_Dividas
FAT_Pagamentos_Faturas
```

Cada tabela possui uma granularidade específica.

Exemplo:

```text
FAT_Dividas
1 linha = 1 dívida

FAT_Parcelas_Dividas
1 linha = 1 parcela

FAT_Pagamentos_Dividas
1 linha = 1 pagamento
```

---

## Principais Regras de Negócio

### A Pagar

Representa uma obrigação financeira do titular.

Exemplos:

- empréstimo pessoal;
- manutenção parcelada;
- compras próprias;
- dívidas fora do cartão.

### A Receber

Representa valores que outras pessoas devem ao titular.

Exemplos:

- cartão emprestado;
- compras realizadas para terceiros;
- despesas compartilhadas.

---

## Cartão Emprestado

Uma das principais regras do projeto é o tratamento de compras realizadas para terceiros.

Exemplo:

```text
Pessoa utiliza o cartão do titular
        ↓
Compra entra na fatura
        ↓
Valor é registrado como A Receber
        ↓
Pessoa realiza o pagamento
        ↓
Saldo a receber é reduzido
```

O valor aparece na fatura do cartão, mas não deve ser interpretado como gasto próprio.

---

## Prevenção de Dupla Contagem

Um cuidado importante foi evitar que parcelas incorporadas às faturas fossem contabilizadas novamente.

Exemplo incorreto:

```text
Fatura = R$ 500
Parcela dentro da fatura = R$ 150

R$ 500 + R$ 150
= dupla contagem
```

A lógica utilizada considera:

```text
Valor Vencido
=
Faturas Vencidas
+
Parcelas Vencidas Fora de Fatura
```

Essa regra também influencia cálculos de fluxo de caixa e compromissos futuros.

---

## Fluxo de Caixa

O projeto diferencia despesa de saída efetiva de dinheiro.

Uma compra realizada no cartão representa uma despesa, mas a saída de caixa acontece somente quando a fatura é paga.

Conceitualmente:

```text
Entradas Caixa
=
Receitas
+
Recebimentos de valores a receber
```

```text
Saídas Caixa
=
Gastos Fora do Cartão
+
Pagamentos de Faturas
+
Pagamentos de Dívidas Fora da Fatura
```

---

## Principais Recursos DAX

O projeto utiliza conceitos como:

- `CALCULATE`
- `FILTER`
- `COALESCE`
- `DIVIDE`
- `USERELATIONSHIP`
- `ALL`
- `ALLSELECTED`
- `REMOVEFILTERS`
- `SELECTEDVALUE`
- variáveis
- funções de data
- contexto de filtro
- contexto temporal

Um dos principais usos de `USERELATIONSHIP` ocorre porque o projeto trabalha com diferentes datas:

- data do gasto;
- data da entrada;
- data de vencimento;
- data do pagamento.

---

## Exemplo de Medida

```DAX
Pagamentos - Faturas =
CALCULATE (
    COALESCE (
        SUM ( FAT_Pagamentos_Faturas[ValorPago] ),
        0
    ),
    USERELATIONSHIP (
        DIM_Calendario[Data],
        FAT_Pagamentos_Faturas[DataPagamento]
    )
)
```

Essa medida utiliza a data real do pagamento para realizar a análise temporal.

---

# Páginas do Dashboard

## Home

Visão executiva da situação financeira.

![Home](assets/dashboard/01_home.webp)

Principais análises:

- receitas;
- despesas;
- fluxo de caixa;
- valores a pagar;
- valores a receber;
- próximos compromissos;
- principais categorias de gastos.

---

## Gastos

Análise detalhada do comportamento das despesas.

![Gastos](assets/dashboard/02_gastos.webp)

Principais análises:

- total de gastos;
- ticket médio;
- evolução mensal;
- categorias;
- frequência dos gastos;
- utilização da meta mensal.

---

## Empréstimos / Valores a Receber

Controle dos valores que outras pessoas devem ao titular.

![Empréstimos](assets/dashboard/03_emprestimos.webp)

Principais análises:

- valor recebido;
- valor pendente;
- situação por pessoa;
- ranking de pendências;
- próximos recebimentos.

---

## Dívidas

Controle das obrigações financeiras próprias.

![Dívidas](assets/dashboard/04_dividas.webp)

Principais análises:

- total das dívidas;
- valor pago;
- saldo devedor;
- status das parcelas;
- alertas;
- compromissos futuros.

---

## Cartões e Faturas

Análise dos cartões e composição das faturas.

![Cartões e Faturas](assets/dashboard/05_cartoes_faturas.webp)

Principais análises:

- saldo por cartão;
- evolução mensal;
- composição da fatura;
- gastos próprios;
- valores de terceiros;
- dívidas pessoais.

---

## Vencimentos

Agenda financeira para acompanhamento de compromissos.

![Vencimentos](assets/dashboard/06_vencimentos.webp)

Principais análises:

- valor pendente;
- faturas vencidas;
- parcelas vencidas;
- próximos vencimentos;
- agenda das faturas.

---

## Pagamentos

Análise dos eventos financeiros efetivamente realizados.

![Pagamentos](assets/dashboard/07_pagamentos.webp)

Principais análises:

- total pago;
- pagamentos de faturas;
- pagamentos de dívidas;
- pagamentos por conta;
- evolução mensal.

---

## Sobre o Projeto

Resumo das tecnologias, competências e funcionalidades aplicadas.

![Sobre o Projeto](assets/dashboard/08_sobre.webp)

---

## Automações com Google Apps Script

O Apps Script funciona como camada de automação operacional.

Entre suas responsabilidades estão:

- processamento dos formulários;
- validações;
- criação de registros;
- criação automática de parcelas;
- associação com cartões;
- associação com faturas;
- atualização de status;
- processamento de pagamentos;
- registros de erro;
- manutenção;
- reprocessamento.

A estrutura foi organizada em módulos:

```text
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
```

---

## Tecnologias Utilizadas

- Power BI
- DAX
- Power Query
- Modelagem Dimensional
- Star Schema
- Google Forms
- Google Sheets
- Google Apps Script
- Git
- GitHub

---

## Competências Aplicadas

- Business Intelligence
- Análise de Dados
- Modelagem de Dados
- Modelagem Dimensional
- Construção de indicadores
- DAX
- Power Query
- ETL
- Automação
- Regras de Negócio
- Data Visualization
- Storytelling com Dados
- UX para BI
- Git e versionamento

---

## Estrutura do Repositório

```text
controle-financeiro-power-bi/
│
├── assets/
│   └── dashboard/
│       ├── 01_home.webp
│       ├── 02_gastos.webp
│       ├── 03_emprestimos.webp
│       ├── 04_dividas.webp
│       ├── 05_cartoes_faturas.webp
│       ├── 06_vencimentos.webp
│       ├── 07_pagamentos.webp
│       └── 08_sobre.webp
│
├── data/
│   └── Controle_Financeiro_Base_Portfolio.xlsx
│
├── docs/
│   ├── 01_VISAO_GERAL.md
│   ├── 02_ARQUITETURA.md
│   ├── 03_MODELO_DE_DADOS.md
│   ├── 04_REGRAS_DE_NEGOCIO.md
│   ├── 05_MEDIDAS_DAX.md
│   ├── 06_AUTOMACOES_APPS_SCRIPT.md
│   ├── 07_DASHBOARD.md
│   ├── DATA_PRIVACY.md
│   └── PROJECT_STATUS.md
│
├── Controle_Financeiro_Portfolio.pbix
├── .gitignore
└── README.md
```

---

# Documentação Técnica

A documentação completa está disponível na pasta [`docs`](docs/).

### 01 - Visão Geral

[Visão Geral do Projeto](docs/01_VISAO_GERAL.md)

Apresenta o problema, objetivo e funcionamento geral da solução.

### 02 - Arquitetura

[Arquitetura da Solução](docs/02_ARQUITETURA.md)

Explica o fluxo entre Forms, Sheets, Apps Script, modelo dimensional e Power BI.

### 03 - Modelo de Dados

[Modelo de Dados](docs/03_MODELO_DE_DADOS.md)

Detalha dimensões, fatos, granularidade e relacionamentos.

### 04 - Regras de Negócio

[Regras de Negócio](docs/04_REGRAS_DE_NEGOCIO.md)

Documenta A Pagar, A Receber, cartão emprestado, faturas, pagamentos e prevenção de dupla contagem.

### 05 - Medidas DAX

[Principais Medidas DAX](docs/05_MEDIDAS_DAX.md)

Apresenta medidas e conceitos analíticos utilizados no projeto.

### 06 - Automações

[Automações com Google Apps Script](docs/06_AUTOMACOES_APPS_SCRIPT.md)

Explica a lógica da camada de automação.

### 07 - Dashboard

[Dashboard](docs/07_DASHBOARD.md)

Documenta todas as páginas do relatório e o fluxo de apresentação.

---

## Arquivo Power BI

O arquivo do dashboard está disponível neste repositório:

```text
Controle_Financeiro_Portfolio.pbix
```

Para visualizar todas as funcionalidades e interações, o arquivo pode ser aberto no **Power BI Desktop**.

---

## Dados do Portfólio

Esta versão pública utiliza **dados fictícios e sanitizados**.

A base pública utilizada para inspeção do modelo está disponível em:

[`data/Controle_Financeiro_Base_Portfolio.xlsx`](data/Controle_Financeiro_Base_Portfolio.xlsx)

O arquivo contém apenas as dimensões e tabelas fato relevantes ao modelo analítico. Abas operacionais de formulários, logs e tabelas auxiliares foram removidas da versão pública.

Os dados preservam:

- estrutura;
- relacionamentos;
- regras de negócio;
- cenários analíticos.

Informações financeiras pessoais reais não fazem parte da versão publicada para portfólio.

Mais informações:

[Política de Privacidade dos Dados](docs/DATA_PRIVACY.md)

---

## Diferenciais do Projeto

O projeto vai além de um dashboard de receitas e despesas.

Entre os principais diferenciais estão:

- integração entre coleta, automação e análise;
- modelagem dimensional;
- controle de cartão emprestado;
- separação entre A Pagar e A Receber;
- controle de parcelas;
- pagamentos parciais;
- decomposição das faturas;
- prevenção de dupla contagem;
- múltiplos contextos de data;
- fluxo de caixa baseado em movimentação financeira real;
- agenda de vencimentos;
- regras de interação entre visuais;
- documentação técnica do projeto.

---

## Possíveis Evoluções

A solução atual utiliza ferramentas acessíveis para construir todo o fluxo.

Uma evolução futura poderia utilizar:

```text
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
```

Outras melhorias possíveis:

- banco de dados relacional;
- pipeline automatizado;
- Data Warehouse;
- atualização automática do Power BI;
- testes automatizados;
- monitoramento das automações;
- API para entrada de movimentações;
- aplicação web ou mobile;
- integração com instituições financeiras.

---

## Autor

Desenvolvido por **Guilherme Mendonça**.

Projeto desenvolvido para aplicação prática, aprendizado e composição de portfólio profissional na área de **Dados e Business Intelligence**.
