# Principais Medidas DAX

## Visão Geral

O projeto utiliza medidas DAX para transformar os dados financeiros em indicadores analíticos.

As medidas foram organizadas por domínio de negócio, incluindo:

- gastos;
- entradas;
- faturas;
- dívidas;
- fluxo de caixa;
- vencimentos;
- limites;
- pagamentos.

Nesta documentação estão algumas das medidas mais relevantes para demonstrar a lógica aplicada no projeto.

O objetivo não é documentar todas as medidas existentes no modelo, mas destacar aquelas que representam as principais decisões de modelagem, contexto de filtro e regras de negócio.

---

# 1. Total de Gastos

```DAX
Total Gastos =
COALESCE (
    SUM ( FAT_Gastos[ValorGasto] ),
    0
)
```

## Objetivo

Calcular o valor total de gastos dentro do contexto atual do relatório.

## Conceitos aplicados

- `SUM`
- `COALESCE`

O `COALESCE` garante retorno igual a zero quando não existem valores no contexto analisado.

---

# 2. Quantidade de Gastos

```DAX
Qtd Gastos =
COUNTROWS ( FAT_Gastos )
```

## Objetivo

Contar a quantidade de registros de gastos existentes no contexto atual.

Essa medida é utilizada, entre outras situações, para cálculo do ticket médio.

---

# 3. Ticket Médio de Gastos

```DAX
Ticket Medio Gasto =
DIVIDE (
    [Total Gastos],
    [Qtd Gastos],
    0
)
```

## Objetivo

Calcular o valor médio das despesas registradas.

## Conceito aplicado

A função `DIVIDE` é utilizada no lugar da divisão direta para tratar situações em que o denominador seja zero.

---

# 4. Total de Entradas

```DAX
Total Entradas =
COALESCE (
    SUM ( FAT_Entradas[ValorEntrada] ),
    0
)
```

## Objetivo

Calcular o total de entradas financeiras.

Essa medida serve como base para indicadores de receita e fluxo de caixa.

---

# 5. Entradas de Caixa

```DAX
Entradas Caixa =
COALESCE ( [Total Entradas], 0 )
+ COALESCE ( [Total Recebido de Dividas], 0 )
```

## Objetivo

Representar o dinheiro efetivamente recebido.

A medida considera:

- receitas;
- recebimentos de valores a receber.

## Regra de negócio

Uma dívida classificada como `A Receber` representa um direito de recebimento.

Ela somente passa a compor efetivamente o caixa quando o pagamento é recebido.

---

# 6. Saídas de Caixa

```DAX
Saidas Caixa =
COALESCE ( [Total Gastos Fora Cartao], 0 )
+ COALESCE ( [Total Pago Faturas], 0 )
+ COALESCE ( [Pago Dividas a Pagar Fora Fatura], 0 )
```

## Objetivo

Representar o dinheiro efetivamente desembolsado.

## Regra de negócio

A medida diferencia:

- gastos que saem diretamente de uma conta;
- pagamentos de faturas;
- pagamentos de dívidas fora de faturas.

Compras realizadas no cartão não devem ser somadas novamente ao fluxo de caixa se o pagamento da fatura já estiver sendo contabilizado.

Essa lógica evita dupla contagem.

---

# 7. Fluxo de Caixa Real

```DAX
Fluxo Caixa Real =
[Entradas Caixa] - [Saidas Caixa]
```

## Objetivo

Calcular o resultado financeiro efetivo entre entradas e saídas de caixa.

Se o resultado for positivo, as entradas foram superiores às saídas.

Se o resultado for negativo, houve maior saída de recursos no período analisado.

---

# 8. Fluxo de Caixa Acumulado

```DAX
Fluxo de Caixa Acumulado =
VAR DataAtual =
    MAX ( DIM_Calendario[Data] )

RETURN
    CALCULATE (
        [Fluxo Caixa Real],
        FILTER (
            ALLSELECTED ( DIM_Calendario[Data] ),
            DIM_Calendario[Data] <= DataAtual
        )
    )
```

## Objetivo

Mostrar a evolução acumulada do fluxo de caixa ao longo do tempo.

## Conceitos aplicados

- variáveis;
- `MAX`;
- `CALCULATE`;
- `FILTER`;
- `ALLSELECTED`;
- contexto temporal.

A medida respeita os filtros aplicados pelo usuário e acumula os valores até a data correspondente ao ponto analisado no visual.

---

# 9. Total de Faturas

```DAX
Total Faturas =
COALESCE (
    CALCULATE (
        SUM ( FAT_Faturas_Cartao[ValorTotalFatura] ),
        USERELATIONSHIP (
            DIM_Calendario[Data],
            FAT_Faturas_Cartao[DataVencimento]
        )
    ),
    0
)
```

## Objetivo

Calcular o valor total das faturas utilizando a data de vencimento como contexto temporal.

## Conceito aplicado

`USERELATIONSHIP`

O modelo possui diferentes tipos de datas.

Por isso, algumas relações com a dimensão calendário permanecem inativas e são ativadas somente dentro da medida correspondente.

---

# 10. Pagamentos de Dívidas por Data de Pagamento

```DAX
Pagamentos - Dívidas =
CALCULATE (
    COALESCE (
        SUM ( FAT_Pagamentos_Dividas[ValorPago] ),
        0
    ),
    USERELATIONSHIP (
        DIM_Calendario[Data],
        FAT_Pagamentos_Dividas[DataPagamento]
    )
)
```

## Objetivo

Calcular pagamentos de dívidas utilizando a data real do pagamento.

## Regra de modelagem

A data de pagamento representa um evento diferente da data de vencimento da parcela.

A utilização de `USERELATIONSHIP` permite analisar cada evento utilizando a mesma dimensão calendário.

---

# 11. Pagamentos de Faturas por Data de Pagamento

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

## Objetivo

Calcular o valor efetivamente pago em faturas utilizando a data do pagamento.

Essa medida é utilizada na página de pagamentos e em análises temporais.

---

# 12. Dívidas a Receber

```DAX
Dividas a Receber =
CALCULATE (
    [Total Parcelas Dividas],
    FAT_Dividas[TipoDivida] = "A Receber"
)
```

## Objetivo

Separar os valores que outras pessoas devem ao titular.

## Regra de negócio

Esses valores representam direitos de recebimento e não devem ser classificados como obrigações próprias.

---

# 13. Total Pago em Dívidas a Pagar

```DAX
Total Pago Dívidas a Pagar =
CALCULATE (
    [Total Pago Dividas],
    FAT_Dividas[TipoDivida] = "A Pagar"
)
```

## Objetivo

Separar os pagamentos relacionados exclusivamente às dívidas próprias.

---

# 14. Pagamentos de Dívidas Fora da Fatura

```DAX
Pago Dividas a Pagar Fora Fatura =
CALCULATE (
    [Total Pago Dívidas a Pagar],
    ISBLANK ( FAT_Parcelas_Dividas[ID_Fatura] )
)
```

## Objetivo

Somar apenas os pagamentos de dívidas pessoais que não estão incorporados a uma fatura de cartão.

## Regra de negócio

Caso uma parcela já esteja dentro de uma fatura, o desembolso financeiro ocorrerá por meio do pagamento daquela fatura.

Somar novamente o pagamento da parcela poderia gerar dupla contagem.

---

# 15. Saldo de Dívidas a Pagar

```DAX
Saldo Dividas a Pagar =
[Dívidas a Pagar]
-
[Total Pago Dívidas a Pagar]
```

## Objetivo

Calcular o valor das obrigações próprias que ainda permanece pendente.

---

# 16. Total Recebido de Dívidas

```DAX
Total Recebido de Dividas =
CALCULATE (
    [Total Pago Dividas],
    FAT_Dividas[TipoDivida] = "A Receber"
)
```

## Objetivo

Calcular o valor efetivamente recebido das dívidas classificadas como `A Receber`.

Esse valor passa a fazer parte das entradas de caixa.

---

# 17. Saldo de Dívidas a Receber

```DAX
Saldo Dividas a Receber =
[Dívidas a Receber]
-
[Total Recebido de Dividas]
```

## Objetivo

Calcular quanto ainda falta receber de terceiros.

---

# 18. Quantidade de Dívidas a Receber Abertas

```DAX
Qtd Dívidas a Receber Abertas =
COALESCE (
    CALCULATE (
        COUNTROWS ( FAT_Dividas ),
        FAT_Dividas[TipoDivida] = "A Receber",
        FAT_Dividas[StatusDivida] <> "Quitada"
    ),
    0
)
```

## Objetivo

Contar apenas as dívidas a receber que ainda possuem algum saldo pendente.

## Regra de negócio

Dívidas quitadas não devem aparecer como compromissos financeiros em aberto.

---

# 19. Percentual de Parcelas Quitadas

```DAX
% Parcelas Quitadas =
DIVIDE (
    [Qtd Parcelas Quitadas],
    [Qtd Total Parcelas],
    0
)
```

## Objetivo

Calcular o percentual de parcelas já quitadas em relação ao total existente.

## Conceito aplicado

A função `DIVIDE` evita erro quando o total de parcelas for zero.

---

# 20. Total Vencido sem Dupla Contagem

```DAX
Total Vencido =
VAR FaturasVencidas =
    COALESCE (
        [Valor Faturas Vencidas],
        0
    )

VAR ParcelasVencidasForaFatura =
    CALCULATE (
        COALESCE (
            SUM ( FAT_Parcelas_Dividas[ValorParcela] ),
            0
        ),
        FAT_Parcelas_Dividas[StatusParcela] = "Vencida",
        ISBLANK ( FAT_Parcelas_Dividas[ID_Fatura] )
    )

RETURN
    FaturasVencidas
        + ParcelasVencidasForaFatura
```

## Objetivo

Calcular o valor financeiro efetivamente vencido sem contabilizar novamente parcelas que já estão incorporadas a uma fatura vencida.

## Regra de negócio

A lógica utilizada é:

```text
Valor Vencido
=
Faturas Vencidas
+
Parcelas Vencidas Fora de Fatura
```

Essa é uma das medidas mais importantes do projeto por resolver diretamente um problema de dupla contagem.

---

# 21. Valor das Faturas nos Próximos 30 Dias

```DAX
Valor Faturas Próximos 30 Dias =
COALESCE (
    CALCULATE (
        SUM ( FAT_Faturas_Cartao[SaldoFatura] ),
        FAT_Faturas_Cartao[StatusFatura] <> "Paga",
        FAT_Faturas_Cartao[DataVencimento] >= TODAY (),
        FAT_Faturas_Cartao[DataVencimento] <= TODAY () + 30
    ),
    0
)
```

## Objetivo

Calcular o saldo das faturas com vencimento dentro dos próximos 30 dias.

## Conceitos aplicados

- `TODAY`;
- filtros condicionais;
- análise prospectiva.

---

# 22. Data da Próxima Fatura com Dívida Pessoal

```DAX
Data Proximo Mes Divida Pessoal Fatura =
CALCULATE (
    MIN ( FAT_Faturas_Cartao[DataVencimento] ),
    FILTER (
        ALL ( FAT_Faturas_Cartao ),
        FAT_Faturas_Cartao[StatusFatura] <> "Paga"
            && COALESCE (
                FAT_Faturas_Cartao[ValorDividasPessoais],
                0
            ) > 0
    )
)
```

## Objetivo

Identificar a próxima fatura ainda não paga que contém valor relacionado a dívida pessoal.

## Conceitos aplicados

- `MIN`;
- `FILTER`;
- `ALL`;
- `COALESCE`;
- filtros condicionais.

---

# 23. Valor da Próxima Dívida Pessoal em Fatura

```DAX
Total Divida Pessoal do Proximo Mes =
VAR DataAlvo =
    [Data Proximo Mes Divida Pessoal Fatura]

VAR InicioMes =
    DATE (
        YEAR ( DataAlvo ),
        MONTH ( DataAlvo ),
        1
    )

VAR FimMes =
    EOMONTH ( DataAlvo, 0 )

RETURN
    IF (
        ISBLANK ( DataAlvo ),
        0,
        CALCULATE (
            SUM ( FAT_Faturas_Cartao[ValorDividasPessoais] ),
            FILTER (
                ALL ( FAT_Faturas_Cartao ),
                FAT_Faturas_Cartao[DataVencimento] >= InicioMes
                    && FAT_Faturas_Cartao[DataVencimento] <= FimMes
                    && FAT_Faturas_Cartao[StatusFatura] <> "Paga"
                    && COALESCE (
                        FAT_Faturas_Cartao[ValorDividasPessoais],
                        0
                    ) > 0
            )
        )
    )
```

## Objetivo

Calcular o valor de dívida pessoal presente na próxima fatura relevante.

## Conceitos aplicados

- variáveis;
- `DATE`;
- `YEAR`;
- `MONTH`;
- `EOMONTH`;
- `FILTER`;
- `ALL`;
- `IF`;
- `ISBLANK`.

---

# 24. Gastos no Cartão no Mês Atual

```DAX
Gastos Cartao Mes Atual =
VAR InicioMesAtual =
    DATE (
        YEAR ( TODAY () ),
        MONTH ( TODAY () ),
        1
    )

VAR FimMesAtual =
    EOMONTH ( TODAY (), 0 )

RETURN
    CALCULATE (
        [Total Gastos],
        REMOVEFILTERS ( DIM_Calendario ),
        FAT_Gastos[DataGasto] >= InicioMesAtual,
        FAT_Gastos[DataGasto] <= FimMesAtual,
        DIM_FormasPagamento[FormaPagamento] = "Crédito"
    )
```

## Objetivo

Calcular os gastos realizados no crédito durante o mês atual.

## Conceitos aplicados

- variáveis;
- `TODAY`;
- `DATE`;
- `EOMONTH`;
- `REMOVEFILTERS`;
- contexto de filtro;
- filtro por dimensão.

---

# 25. Meta Mensal de Gastos

```DAX
Meta Gastos Mensal =
1500
```

## Objetivo

Definir uma referência mensal para acompanhamento dos gastos.

Essa medida é utilizada como base para o indicador de utilização da meta.

---

# 26. Percentual da Meta de Gastos Utilizada

```DAX
% Meta Gastos Usada =
DIVIDE (
    [Gastos Cartao Mes Atual],
    [Meta Gastos Mensal],
    0
)
```

## Objetivo

Mostrar qual percentual da meta mensal de gastos já foi utilizado.

---

# 27. Limite Total dos Cartões

```DAX
Limite Total Cartoes =
SUM ( DIM_Cartoes[LimiteTotal] )
```

## Objetivo

Somar os limites cadastrados dos cartões no contexto atual.

---

# 28. Limite Usado Estimado

```DAX
Limite Usado Estimado =
CALCULATE (
    SUM ( FAT_Faturas_Cartao[SaldoFatura] ),
    FAT_Faturas_Cartao[StatusFatura] <> "Paga"
)
```

## Objetivo

Estimar o valor atualmente comprometido com faturas ainda não pagas.

## Observação

O indicador é tratado como estimativa porque o saldo das faturas não necessariamente representa exatamente o limite utilizado informado em tempo real pela instituição financeira.

---

# 29. Limite Disponível Estimado

```DAX
Limite Disponivel Estimado =
[Limite Total Cartoes]
-
[Limite Usado Estimado]
```

## Objetivo

Estimar o limite ainda disponível nos cartões.

---

# 30. Percentual de Limite Usado

```DAX
% Limite Usado =
DIVIDE (
    [Limite Usado Estimado],
    [Limite Total Cartoes],
    0
)
```

## Objetivo

Representar percentualmente quanto do limite total cadastrado está comprometido.

---

# 31. Scroller de Próximas Faturas

```DAX
Valor Scroller Faturas =
VAR DiaSelecionado =
    SELECTEDVALUE (
        TABELA_Scroller_Faturas[DiaVencimento]
    )

VAR ProximaDataVencimento =
    CALCULATE (
        MIN (
            FAT_Faturas_Cartao[DataVencimento]
        ),
        REMOVEFILTERS (
            DIM_Calendario
        ),
        FILTER (
            ALL (
                FAT_Faturas_Cartao
            ),
            DAY ( FAT_Faturas_Cartao[DataVencimento] ) = DiaSelecionado
                && FAT_Faturas_Cartao[StatusFatura] = "Aberto"
                && FAT_Faturas_Cartao[SaldoFatura] > 0
        )
    )

RETURN
    COALESCE (
        CALCULATE (
            [Total Fatura Sem Emprestimos],
            REMOVEFILTERS (
                DIM_Calendario
            ),
            FAT_Faturas_Cartao[DataVencimento] = ProximaDataVencimento
        ),
        0
    )
```

## Objetivo

Encontrar dinamicamente a próxima fatura em aberto para determinado dia de vencimento.

A medida apresenta o valor da obrigação pessoal sem incorporar valores relacionados ao cartão emprestado.

## Conceitos aplicados

- `SELECTEDVALUE`;
- variáveis;
- `MIN`;
- `FILTER`;
- `ALL`;
- `REMOVEFILTERS`;
- `DAY`;
- contexto dinâmico.

---

# 32. Separação entre Gastos Próprios e Cartão Emprestado

Uma das características importantes do modelo é a existência de medidas que diferenciam o valor total da fatura do valor efetivamente pertencente ao titular.

Conceitualmente:

```text
Fatura Total
=
Gastos Próprios
+
Dívidas Pessoais
+
Valores de Cartão Emprestado
+
Outros Componentes
```

Para determinadas análises, principalmente próximas obrigações pessoais, os valores de terceiros precisam ser retirados.

Essa separação evita interpretar como gasto pessoal um valor que deverá ser recuperado posteriormente.

---

# Contexto de Filtro

Um dos principais conceitos aplicados no projeto é o contexto de filtro.

Uma mesma medida pode apresentar valores diferentes dependendo dos filtros utilizados no relatório.

Exemplos de filtros existentes:

- pessoa;
- cartão;
- categoria;
- mês;
- conta;
- status da parcela.

O DAX utiliza esse contexto para recalcular os indicadores dinamicamente.

---

# Contexto Temporal

O projeto possui diferentes tipos de datas, incluindo:

- data do gasto;
- data da entrada;
- data do vencimento;
- data do pagamento;
- data de abertura da fatura;
- data de fechamento.

Essas datas representam eventos distintos.

Por isso, algumas medidas precisam definir explicitamente qual data será utilizada durante a análise.

---

# Uso de USERELATIONSHIP

O projeto utiliza relacionamentos inativos com a dimensão calendário.

Exemplo:

```DAX
USERELATIONSHIP (
    DIM_Calendario[Data],
    FAT_Pagamentos_Dividas[DataPagamento]
)
```

Essa técnica permite utilizar a mesma `DIM_Calendario` em diferentes análises sem criar múltiplas dimensões de data para cada evento.

---

# Uso de REMOVEFILTERS

Algumas medidas precisam ignorar temporariamente o filtro de calendário aplicado na página.

Exemplo:

```DAX
REMOVEFILTERS ( DIM_Calendario )
```

Esse recurso é utilizado quando uma medida precisa localizar uma informação global, como a próxima fatura disponível, independentemente do filtro temporal aplicado ao visual.

---

# Uso de ALL

A função `ALL` é utilizada em algumas medidas para retirar filtros existentes de uma tabela durante uma busca específica.

Exemplo:

```DAX
ALL ( FAT_Faturas_Cartao )
```

Isso permite procurar registros em toda a tabela antes de aplicar novamente as condições desejadas.

---

# Uso de ALLSELECTED

A função `ALLSELECTED` é utilizada no fluxo de caixa acumulado.

Ela permite manter os filtros selecionados pelo usuário enquanto remove o contexto específico de cada ponto do gráfico para realizar o cálculo acumulado.

---

# Uso de COALESCE

A função `COALESCE` aparece em diversas medidas.

Exemplo:

```DAX
COALESCE (
    [Total Entradas],
    0
)
```

Seu objetivo é transformar valores em branco em zero quando isso melhora a interpretação do indicador.

---

# Uso de DIVIDE

A função `DIVIDE` é utilizada principalmente em indicadores percentuais.

Exemplo:

```DAX
DIVIDE (
    [Qtd Parcelas Quitadas],
    [Qtd Total Parcelas],
    0
)
```

Essa função oferece tratamento seguro para divisão por zero.

---

# Uso de Variáveis

As variáveis são utilizadas para deixar medidas mais legíveis e evitar repetição de cálculos.

Exemplo:

```DAX
VAR InicioMesAtual =
    DATE (
        YEAR ( TODAY () ),
        MONTH ( TODAY () ),
        1
    )
```

As variáveis também facilitam manutenção e depuração das medidas.

---

# Organização das Medidas

As medidas foram organizadas por domínio de negócio para facilitar manutenção e leitura do modelo.

Os principais grupos utilizados são:

```text
02 - Gastos
03 - Entradas
04 - Faturas
05 - Dívidas
06 - Fluxo de Caixa
07 - Vencimentos
08 - Limites
09 - Pagamentos
```

Essa organização permite localizar rapidamente as medidas relacionadas ao mesmo contexto financeiro.

---

# Decisões de Desenvolvimento

Durante o desenvolvimento, diversas medidas foram revisadas após testes de consistência e comportamento dos filtros.

Entre os principais pontos analisados estiveram:

- prevenção de dupla contagem;
- diferenciação entre posição financeira e movimentação financeira;
- escolha da data correta para cada análise;
- utilização de relacionamentos ativos e inativos;
- comportamento de segmentadores;
- tratamento de valores em branco;
- separação entre gastos próprios e valores de terceiros;
- cálculo de compromissos futuros;
- cálculo de valores vencidos;
- cálculo de fluxo de caixa real.

Essas revisões fizeram parte do processo de validação do modelo.

---

# Posição Financeira x Movimento Financeiro

Uma decisão importante do projeto foi distinguir medidas que representam posição financeira de medidas que representam eventos ocorridos durante determinado período.

## Posição financeira

Exemplos:

- saldo de dívida;
- saldo de fatura;
- valor pendente;
- próxima parcela;
- próximo compromisso.

Esses indicadores representam um estado.

## Movimento financeiro

Exemplos:

- pagamento realizado;
- receita recebida;
- gasto realizado;
- recebimento de dívida.

Esses indicadores representam eventos ocorridos no tempo.

Essa diferenciação também influenciou a configuração das interações dos filtros do dashboard.

---

# DAX e Regra de Negócio

As medidas DAX do projeto não foram utilizadas somente para realizar somas.

Parte das regras de negócio financeiras está implementada diretamente nas medidas.

Exemplos:

- diferenciação entre `A Pagar` e `A Receber`;
- exclusão de parcelas já incorporadas às faturas;
- cálculo de pagamentos pela data real de pagamento;
- separação entre gasto no cartão e saída de caixa;
- cálculo de saldo;
- cálculo de compromissos futuros;
- exclusão de valores de cartão emprestado em determinadas análises.

---

# Principais Desafios Tratados com DAX

Os principais desafios analíticos tratados foram:

1. trabalhar com múltiplos tipos de data;
2. evitar dupla contagem;
3. diferenciar obrigação financeira de pagamento;
4. calcular saldo de valores a pagar e a receber;
5. separar valores próprios de valores de terceiros;
6. controlar contexto de filtro;
7. analisar compromissos futuros;
8. representar pagamentos parciais;
9. calcular fluxo de caixa real;
10. permitir análises temporais consistentes.

---

# Resumo Técnico

O DAX foi utilizado como camada de lógica analítica do projeto.

As medidas trabalham em conjunto com:

- o modelo dimensional;
- os relacionamentos;
- as regras de negócio;
- a dimensão calendário;
- os filtros do relatório.

O objetivo foi construir indicadores que representassem corretamente a situação financeira e não apenas somassem os valores armazenados nas tabelas.

A combinação entre modelagem dimensional, contexto de filtro e medidas DAX tornou possível representar cenários como:

- compras no cartão;
- dívidas pessoais;
- cartão emprestado;
- valores a receber;
- parcelas;
- pagamentos parciais;
- faturas vencidas;
- compromissos futuros;
- fluxo de caixa;
- prevenção de dupla contagem.
