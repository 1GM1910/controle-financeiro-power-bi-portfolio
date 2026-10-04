# Regras de Negócio

## Visão Geral

As regras de negócio são a base da modelagem, das automações e das medidas DAX.

O projeto trata gastos próprios, cartão emprestado, dívidas, valores a pagar e a receber, parcelas, pagamentos parciais, faturas, vencimentos e fluxo de caixa.

## 1. A Pagar e A Receber

### A Pagar

Representa obrigação financeira do titular.

Exemplos:

- empréstimo pessoal;
- manutenção parcelada;
- curso parcelado;
- compra própria parcelada.

### A Receber

Representa valor devido por terceiros ao titular.

Exemplos:

- cartão emprestado;
- compra para outra pessoa;
- despesa compartilhada.

Valores a receber não devem ser tratados como obrigação própria.

## 2. Cartão Emprestado

~~~text
Compra para terceiro
      ↓
Valor entra na fatura do titular
      ↓
Dívida A Receber é criada
      ↓
Pessoa realiza pagamento
      ↓
Saldo a receber é reduzido
~~~

O valor faz parte da fatura, mas não representa necessariamente gasto próprio.

## 3. Composição da Fatura

Uma fatura pode combinar:

~~~text
Gastos próprios
+
Cartão emprestado
+
Dívidas pessoais
+
Outros componentes
~~~

Essa separação permite distinguir obrigação total com o banco de responsabilidade financeira própria.

## 4. Dívida, Parcela e Pagamento

~~~text
Dívida
  ↓
Parcela
  ↓
Pagamento
~~~

São eventos diferentes:

- dívida = compromisso completo;
- parcela = vencimento individual;
- pagamento = evento financeiro.

## 5. Parcelamento

Uma dívida de R$ 1.200 em quatro parcelas pode gerar quatro registros de R$ 300, cada um com número, vencimento e status próprios.

## 6. Status das Parcelas

Estados utilizados incluem:

- Quitada;
- Vencida;
- Aberta;
- Parcial.

Uma parcela parcial possui pagamento registrado, mas ainda mantém saldo.

## 7. Pagamentos Parciais

~~~text
Parcela = R$ 300
Pagamento = R$ 160
Saldo = R$ 140
Status = Parcial
~~~

O valor restante continua pendente.

## 8. Fatura e Pagamento da Fatura

A fatura representa uma obrigação. O pagamento representa a saída efetiva de caixa.

Por isso, fatura e pagamento são armazenados separadamente.

## 9. Status das Faturas

Exemplos:

- Paga;
- Parcial;
- Aberto;
- Vencida.

O status depende de saldo, pagamentos e vencimento.

## 10. Fluxo de Caixa

~~~text
Despesa
≠
Saída imediata de caixa
~~~

Uma compra no cartão é uma despesa, mas a saída de dinheiro acontece quando a fatura é paga.

## 11. Entradas de Caixa

~~~text
Entradas de Caixa
=
Receitas
+
Recebimentos de Dívidas
~~~

Um valor A Receber só entra no caixa quando é efetivamente recebido.

## 12. Saídas de Caixa

~~~text
Saídas de Caixa
=
Gastos Fora do Cartão
+
Pagamentos de Faturas
+
Pagamentos de Dívidas Fora da Fatura
~~~

## 13. Prevenção de Dupla Contagem

Se uma parcela de R$ 300 já está dentro de uma fatura de R$ 500, somar R$ 500 + R$ 300 seria incorreto.

O compromisso real já está representado na fatura.

## 14. Total Vencido

~~~text
Total Vencido
=
Faturas Vencidas
+
Parcelas Vencidas Fora de Fatura
~~~

Parcelas vinculadas a faturas não são somadas novamente.

## 15. Dívidas Fora da Fatura

A mesma lógica é usada no fluxo de caixa: pagamentos de dívidas entram diretamente apenas quando a parcela não está vinculada a uma fatura.

## 16. Datas Diferentes para Eventos Diferentes

O modelo distingue:

- data do gasto;
- data da entrada;
- data de início da dívida;
- data de vencimento;
- data de pagamento.

Data de vencimento responde a perguntas de compromisso. Data de pagamento responde a perguntas de caixa realizado.

## 17. Posição Financeira x Movimento Financeiro

### Posição

Exemplos:

- saldo de dívida;
- saldo de fatura;
- valor pendente;
- próxima parcela.

### Movimento

Exemplos:

- receita recebida;
- gasto realizado;
- pagamento de dívida;
- pagamento de fatura.

Essa diferença influencia filtros e interações do dashboard.

## 18. Filtros de Mês

Nem todos os indicadores de posição devem responder ao filtro de mês. Algumas interações foram desativadas intencionalmente para preservar o significado do indicador.

## 19. Filtro de Pessoa

Em algumas páginas, o filtro de pessoa atua principalmente sobre dívidas e valores a receber. Aplicá-lo sobre todas as faturas poderia gerar interpretação incorreta.

## 20. Próximos Compromissos

Medidas de próximos vencimentos consideram apenas obrigações ainda não quitadas.

## 21. Limite Utilizado

~~~text
Limite Utilizado Estimado
=
Saldo das Faturas Não Pagas
~~~

O indicador é uma estimativa, pois o limite oficial depende das regras da instituição financeira.

## 22. Dados de Portfólio

A versão pública utiliza dados fictícios e sanitizados, preservando cenários como dívida quitada, dívida parcial, dívida vencida, cartão emprestado, pagamentos parciais e faturas em diferentes status.

## 23. Preservação do Histórico

Pagamentos não substituem parcelas ou faturas originais; são registrados como eventos relacionados. Isso preserva rastreabilidade.

## Resumo

~~~text
A Pagar       = obrigação do titular
A Receber     = valor devido por terceiros
Cartão Emprestado = fatura + A Receber
Dívida ≠ Parcela ≠ Pagamento
Compra no cartão ≠ saída imediata de caixa
Pagamento da fatura = saída de caixa
Parcela dentro da fatura não é somada novamente
Data de vencimento ≠ Data de pagamento
Posição financeira ≠ Movimento financeiro
~~~

O objetivo das regras é manter consistência entre obrigação, vencimento, pagamento, recebimento, saldo e fluxo de caixa.
