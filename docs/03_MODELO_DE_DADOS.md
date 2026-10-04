# Modelo de Dados

## Visão Geral

O projeto utiliza Modelagem Dimensional em formato Star Schema.

A principal ideia é separar:

- dimensões, que descrevem o contexto;
- fatos, que registram os eventos financeiros.

Essa estrutura reduz ambiguidades e facilita as medidas DAX.

## Dimensões

### DIM_Calendario

Centraliza atributos temporais como Data, Ano, Mês, Ano/Mês, Dia e Dia da Semana.

### DIM_Pessoas

Armazena pessoas relacionadas às movimentações, incluindo titular e pessoas associadas a valores a receber.

### DIM_Cartoes

Armazena cartões, instituição, limite e atributos utilizados nas análises de faturas e compromissos.

### DIM_Categorias

Classifica os gastos por finalidade e pode indicar características como essencial ou não essencial.

### DIM_Contas

Representa as contas utilizadas nas entradas e saídas financeiras.

### DIM_FormasPagamento

Classifica formas como Pix, Crédito, Débito e Dinheiro.

## Tabelas Fato

### FAT_Gastos

Granularidade:

~~~text
1 linha = 1 gasto
~~~

Exemplos de campos: ID_Gasto, DataGasto, ID_Pessoa, ID_Categoria, ID_FormaPagamento, ID_Conta, ID_Cartao, ID_Fatura, Descricao, ValorGasto e TipoGasto.

### FAT_Entradas

Granularidade:

~~~text
1 linha = 1 entrada financeira
~~~

Registra salário, freelance, reembolso e outras entradas.

### FAT_Dividas

Granularidade:

~~~text
1 linha = 1 dívida
~~~

Exemplos de campos: ID_Divida, ID_Pessoa, TipoDivida, OrigemDivida, DescricaoDivida, DataInicio, ValorTotal, QtdeParcelas, ID_Cartao e StatusDivida.

### FAT_Parcelas_Dividas

Granularidade:

~~~text
1 linha = 1 parcela
~~~

Registra número da parcela, vencimento, valor, cartão, fatura e status.

### FAT_Faturas_Cartao

Granularidade:

~~~text
1 linha = 1 fatura
~~~

A tabela separa ValorGastosProprios, ValorDividasCartaoEmprestado, ValorDividasPessoais, ValorTotalFatura, ValorPagoFatura e SaldoFatura.

### FAT_Pagamentos_Dividas

Granularidade:

~~~text
1 linha = 1 pagamento de parcela
~~~

Permite representar pagamentos parciais ou múltiplos pagamentos sobre uma obrigação.

### FAT_Pagamentos_Faturas

Granularidade:

~~~text
1 linha = 1 pagamento de fatura
~~~

Separa a obrigação da fatura do evento de pagamento.

## Dívida, Parcela e Pagamento

~~~text
Dívida
  ↓
Parcelas
  ↓
Pagamentos
~~~

A dívida representa o compromisso completo, a parcela representa cada vencimento e o pagamento registra o evento financeiro.

## Fatura e Pagamento

~~~text
Fatura
   ↓
Pagamento da Fatura
~~~

Uma fatura pode existir sem pagamento e pode receber pagamentos parciais.

## Relacionamentos

O modelo prioriza relações 1:* com filtro da dimensão para a fato.

~~~text
DIM_Pessoas
     1
     |
     *
FAT_Dividas
~~~

Relacionamentos muitos-para-muitos são evitados sempre que possível.

## Relacionamentos Temporais

A DIM_Calendario é utilizada com diferentes datas, como:

- FAT_Gastos[DataGasto];
- FAT_Entradas[DataEntrada];
- FAT_Faturas_Cartao[DataVencimento];
- FAT_Parcelas_Dividas[DataVencimento];
- FAT_Pagamentos_Dividas[DataPagamento];
- FAT_Pagamentos_Faturas[DataPagamento].

Algumas relações ficam inativas e são ativadas por medidas com USERELATIONSHIP.

## Chaves

O modelo utiliza IDs estáveis como ID_Pessoa, ID_Cartao, ID_Categoria, ID_Conta, ID_Divida, ID_ParcelaDivida e ID_Fatura.

## Cartão Emprestado

~~~text
Pessoa
   ↓
Dívida A Receber
   ↓
Parcela
   ↓
Fatura
~~~

A fatura continua pertencendo ao titular, enquanto a dívida registra o valor a receber de terceiro.

## Composição da Fatura

A fatura pode combinar:

- gastos próprios;
- cartão emprestado;
- dívidas pessoais;
- outros componentes.

Essa separação permite analisar a responsabilidade financeira real do titular.

## Resumo de Granularidade

~~~text
FAT_Gastos                → 1 linha = 1 gasto
FAT_Entradas              → 1 linha = 1 entrada
FAT_Dividas               → 1 linha = 1 dívida
FAT_Parcelas_Dividas      → 1 linha = 1 parcela
FAT_Faturas_Cartao        → 1 linha = 1 fatura
FAT_Pagamentos_Dividas    → 1 linha = 1 pagamento de parcela
FAT_Pagamentos_Faturas    → 1 linha = 1 pagamento de fatura
~~~

## Benefícios

A modelagem dimensional melhora clareza, manutenção, consistência dos filtros e organização das medidas DAX.

## Resumo Técnico

Os principais conceitos aplicados foram Star Schema, dimensões, fatos, granularidade, chaves, relacionamentos 1:*, direção de filtro simples, dimensão calendário e separação entre obrigação e pagamento.
