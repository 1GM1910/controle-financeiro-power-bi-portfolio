# Automações com Google Apps Script

## Visão Geral

O projeto utiliza Google Apps Script como camada de automação e processamento entre a entrada dos dados no Google Forms, o armazenamento no Google Sheets e o consumo analítico no Power BI.

O objetivo dessa camada é reduzir atividades manuais, aplicar regras de negócio de forma consistente e manter as tabelas financeiras organizadas.

A arquitetura simplificada é:

```text
Google Forms
      ↓
Google Sheets
      ↓
Google Apps Script
      ↓
Tabelas estruturadas
      ↓
Power BI
```

O Apps Script atua principalmente no processamento das movimentações financeiras após o registro dos dados.

---

# Objetivos da Automação

As automações foram desenvolvidas para:

- processar respostas dos formulários;
- validar os dados recebidos;
- gerar registros estruturados;
- criar parcelas;
- associar movimentações a cartões;
- associar movimentações a faturas;
- registrar pagamentos;
- atualizar status;
- tratar dívidas a pagar;
- tratar valores a receber;
- controlar cartão emprestado;
- registrar logs;
- tratar erros;
- permitir reprocessamento de registros;
- reduzir edição manual das tabelas.

---

# Organização do Código

O código do Apps Script foi separado em módulos com responsabilidades específicas.

A estrutura utilizada foi organizada da seguinte forma:

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

Essa separação facilita:

- leitura do código;
- manutenção;
- testes;
- localização de funções;
- evolução futura da solução.

---

# 1. Configuração

Arquivo:

```text
00_Config.gs
```

## Responsabilidade

Centralizar configurações utilizadas pelo restante do projeto.

Exemplos de configurações:

- nomes de abas;
- parâmetros do sistema;
- constantes;
- referências utilizadas por diferentes módulos.

## Benefício

Evita espalhar valores fixos por todo o código.

Quando uma configuração precisa ser alterada, a mudança pode ser feita em um ponto central.

---

# 2. Funções Utilitárias

Arquivo:

```text
01_Utils.gs
```

## Responsabilidade

Armazenar funções genéricas reutilizadas por diferentes partes do projeto.

Exemplos conceituais:

- conversão de valores;
- manipulação de datas;
- localização de registros;
- formatação;
- tratamento de campos;
- funções auxiliares de leitura e escrita.

## Benefício

Evita repetição de código e melhora a reutilização.

---

# 3. Logs

Arquivo:

```text
02_Logs.gs
```

## Responsabilidade

Registrar informações importantes sobre a execução das automações.

Os logs podem ser utilizados para registrar:

- processamento concluído;
- registros processados;
- erros;
- mensagens de validação;
- manutenção;
- reprocessamentos.

## Objetivo

Aumentar a rastreabilidade da solução.

Caso algum problema aconteça, os registros de execução ajudam a identificar em qual etapa ocorreu a falha.

---

# 4. Validações

Arquivo:

```text
03_Validacoes.gs
```

## Responsabilidade

Verificar se os dados recebidos estão em condições adequadas antes de serem processados.

Exemplos de validações:

- campos obrigatórios;
- existência de pessoa;
- existência de cartão;
- existência de conta;
- valor válido;
- número de parcelas;
- situação da movimentação;
- consistência das referências.

## Objetivo

Evitar que dados incompletos ou inválidos gerem registros incorretos nas tabelas estruturadas.

---

# 5. Listas de Apoio

Arquivo:

```text
04_ListasApoio.gs
```

## Responsabilidade

Auxiliar na manutenção de listas e valores utilizados pelos formulários e pela base operacional.

Essas listas podem ser relacionadas a:

- pessoas;
- cartões;
- contas;
- categorias;
- formas de pagamento;
- status;
- opções auxiliares.

## Objetivo

Manter os valores utilizados na entrada de dados alinhados com os cadastros existentes.

---

# 6. Faturas

Arquivo:

```text
05_Faturas.gs
```

## Responsabilidade

Concentrar as regras relacionadas às faturas de cartão.

Entre as responsabilidades estão:

- localizar a fatura correspondente;
- relacionar compras à fatura;
- atualizar os valores da fatura;
- calcular componentes da fatura;
- atualizar saldo;
- atualizar status;
- tratar pagamentos;
- separar gastos próprios e valores de terceiros.

---

# Associação com Fatura

Uma compra no cartão precisa ser associada à fatura correta.

A lógica depende de informações como:

- cartão;
- data da movimentação;
- período de fechamento;
- vencimento.

Exemplo conceitual:

```text
Compra no cartão
      ↓
Identificação do cartão
      ↓
Identificação do período
      ↓
Localização da fatura
      ↓
Associação do ID_Fatura
```

Essa referência é importante para que o Power BI consiga analisar corretamente gastos, parcelas e faturas.

---

# Atualização da Fatura

A fatura pode possuir diferentes componentes:

```text
Fatura
├── Gastos próprios
├── Dívidas pessoais
├── Valores de cartão emprestado
└── Outros componentes
```

A automação atualiza os valores de acordo com os registros associados àquela fatura.

---

# Status da Fatura

O status pode variar conforme:

- valor total;
- valor pago;
- saldo;
- vencimento.

Exemplos:

```text
Paga
Parcial
Aberto
Vencida
```

A automação ajuda a manter o status coerente com a situação financeira da fatura.

---

# 7. Dívidas

Arquivo:

```text
06_Dividas.gs
```

## Responsabilidade

Concentrar o processamento relacionado a dívidas e valores a receber.

Entre as responsabilidades estão:

- criação da dívida;
- identificação da pessoa;
- identificação do tipo;
- definição da origem;
- criação das parcelas;
- relacionamento com cartão;
- relacionamento com fatura;
- atualização de status.

---

# Tipos de Dívida

O projeto diferencia principalmente:

```text
A Pagar
A Receber
```

## A Pagar

Representa uma obrigação financeira própria.

## A Receber

Representa um valor que outra pessoa deve ao titular.

Essa distinção é fundamental para o funcionamento do modelo financeiro.

---

# Origem da Dívida

As dívidas também podem possuir diferentes origens.

Entre os cenários tratados estão:

```text
DividaPessoal
CartaoEmprestado
```

## DividaPessoal

Representa uma obrigação do próprio titular.

## CartaoEmprestado

Representa uma compra realizada no cartão do titular para outra pessoa.

Nesse cenário, a fatura pertence ao titular, mas o valor correspondente deve ser recebido da outra pessoa.

---

# Criação de Parcelas

Quando uma dívida é parcelada, o sistema cria registros individuais na tabela de parcelas.

Exemplo:

```text
Dívida de R$ 1.200
Quantidade de parcelas = 4
        ↓
Parcela 1 = R$ 300
Parcela 2 = R$ 300
Parcela 3 = R$ 300
Parcela 4 = R$ 300
```

Cada parcela possui sua própria:

- numeração;
- data de vencimento;
- valor;
- status;
- referência ao cartão;
- referência à fatura quando aplicável.

---

# Benefício da Separação entre Dívida e Parcela

Essa estrutura permite que:

- uma dívida possua múltiplos vencimentos;
- parcelas sejam pagas separadamente;
- parcelas tenham status diferentes;
- o histórico seja preservado;
- compromissos futuros sejam analisados.

---

# 8. Pagamentos

Arquivo:

```text
07_Pagamentos.gs
```

## Responsabilidade

Processar pagamentos relacionados a:

- parcelas de dívidas;
- faturas de cartão.

O pagamento é tratado como evento separado da obrigação original.

---

# Pagamento de Dívida

Fluxo conceitual:

```text
Pagamento registrado
      ↓
Identificação da parcela
      ↓
Registro do valor pago
      ↓
Atualização da parcela
      ↓
Atualização da dívida
```

O sistema pode atualizar:

- valor pago;
- status da parcela;
- status da dívida;
- saldo pendente.

---

# Pagamento Parcial

A automação permite representar pagamentos inferiores ao valor total da obrigação.

Exemplo:

```text
Parcela = R$ 300
Pagamento = R$ 160
Saldo = R$ 140
```

Nesse cenário, a obrigação não pode ser considerada totalmente quitada.

---

# Pagamento de Fatura

Fluxo conceitual:

```text
Pagamento de fatura
      ↓
Identificação da fatura
      ↓
Registro do pagamento
      ↓
Atualização do valor pago
      ↓
Atualização do saldo
      ↓
Atualização do status
```

---

# 9. Processamento dos Formulários

Arquivo:

```text
08_Forms.gs
```

## Responsabilidade

Receber as informações originadas nos formulários e direcionar cada registro para o fluxo correspondente.

Exemplos:

```text
Entrada
Gasto
Dívida
Pagamento de Dívida
Pagamento de Fatura
```

O processamento identifica o tipo de movimentação e chama as funções específicas responsáveis pela operação.

---

# Fluxo Simplificado do Formulário

```text
Usuário envia o formulário
        ↓
Trigger detecta nova resposta
        ↓
Apps Script lê os campos
        ↓
Valida os dados
        ↓
Identifica o tipo de movimentação
        ↓
Executa o módulo correspondente
        ↓
Atualiza as tabelas
        ↓
Registra o processamento
```

---

# 10. Manutenção

Arquivo:

```text
09_Manutencao.gs
```

## Responsabilidade

Armazenar funções utilizadas para manutenção e correção da base.

Exemplos de uso:

- reprocessamento;
- correção de registros;
- reconstrução de informações;
- atualização de status;
- apoio em testes.

---

# Reprocessamento

Durante o desenvolvimento foi criada a possibilidade de reprocessar registros.

Isso é útil quando:

- uma automação falha;
- uma regra de negócio é corrigida;
- um registro precisa ser recalculado;
- uma resposta precisa ser processada novamente.

Fluxo conceitual:

```text
Registro com problema
      ↓
Correção da regra
      ↓
Reprocessamento
      ↓
Registro atualizado
```

Essa funcionalidade evita a necessidade de recriar manualmente toda a movimentação.

---

# 11. Triggers

Arquivo:

```text
10_Triggers.gs
```

## Responsabilidade

Controlar os gatilhos responsáveis por iniciar as automações.

Um trigger pode executar uma função automaticamente quando determinado evento acontece.

Exemplo:

```text
Novo formulário enviado
        ↓
Trigger executado
        ↓
Função de processamento chamada
```

Isso permite que o sistema funcione sem necessidade de executar manualmente cada processamento.

---

# 12. Formatação

Arquivo:

```text
11_Formatacao.gs
```

## Responsabilidade

Aplicar formatações e padronizações na base operacional.

Exemplos:

- datas;
- moedas;
- cabeçalhos;
- organização visual;
- padronização de tabelas.

## Objetivo

Melhorar a legibilidade e manter consistência visual no Google Sheets.

---

# Tratamento de Erros

Uma automação financeira precisa prever que erros podem acontecer.

Por isso, o projeto possui mecanismos para registrar falhas e permitir análise posterior.

Exemplos de problemas que podem ocorrer:

- campo ausente;
- referência inexistente;
- cartão não localizado;
- fatura não localizada;
- bloqueio temporário da planilha;
- dados inconsistentes;
- erro de processamento.

---

# Estratégia de Logs de Erro

Fluxo conceitual:

```text
Automação executada
      ↓
Erro detectado
      ↓
Erro registrado
      ↓
Processamento interrompido ou tratado
      ↓
Registro analisado posteriormente
```

Essa abordagem melhora a capacidade de manutenção da solução.

---

# Concorrência e Bloqueios

Como diferentes automações podem tentar acessar a planilha, podem existir cenários de concorrência.

Durante o desenvolvimento foram encontrados casos relacionados a bloqueio de execução.

Esse tipo de situação exige controle para evitar que duas operações alterem os mesmos dados ao mesmo tempo.

O tratamento de concorrência é importante principalmente em automações que:

- escrevem em tabelas;
- atualizam faturas;
- alteram parcelas;
- registram pagamentos.

---

# Separação de Responsabilidades

A divisão em arquivos separados segue o princípio de separação de responsabilidades.

Cada módulo deve tratar um grupo específico de funções.

Exemplo:

```text
Faturas      → regras de fatura
Dívidas      → regras de dívida
Pagamentos   → regras de pagamento
Forms        → entrada
Logs         → rastreabilidade
Validações   → consistência
```

Essa organização facilita a manutenção e reduz o risco de criar um único arquivo grande e difícil de entender.

---

# Relação entre Apps Script e Power BI

O Apps Script pertence à camada operacional.

O Power BI pertence à camada analítica.

A função do Apps Script é preparar e manter os dados estruturados.

A função do Power BI é analisar esses dados.

Fluxo:

```text
Apps Script
      ↓
Dados estruturados
      ↓
Power BI
      ↓
Indicadores
```

O Apps Script não substitui o DAX.

Cada tecnologia possui uma responsabilidade diferente.

---

# Apps Script x DAX

## Apps Script

Utilizado para:

- processar registros;
- alterar dados;
- criar parcelas;
- atualizar tabelas;
- validar informações;
- automatizar processos.

## DAX

Utilizado para:

- calcular indicadores;
- controlar contexto de análise;
- calcular saldos;
- criar métricas;
- realizar análises temporais.

Essa separação permite manter lógica operacional e lógica analítica em camadas diferentes.

---

# Exemplo Completo: Cartão Emprestado

Um dos fluxos mais importantes do projeto é o cartão emprestado.

Exemplo:

```text
Pessoa utiliza cartão do titular
        ↓
Compra registrada
        ↓
Cartão identificado
        ↓
Fatura correspondente localizada
        ↓
Valor incluído na fatura
        ↓
Dívida A Receber criada
        ↓
Parcelas criadas
        ↓
Pessoa realiza pagamento
        ↓
Pagamento registrado
        ↓
Saldo A Receber atualizado
```

Esse fluxo representa dois compromissos financeiros diferentes:

```text
Titular → deve pagar a fatura ao banco

Outra pessoa → deve pagar o valor ao titular
```

A automação mantém essas duas obrigações separadas.

---

# Exemplo Completo: Dívida Pessoal

Outro fluxo importante é a dívida pessoal.

```text
Dívida registrada
      ↓
Tipo = A Pagar
      ↓
Parcelas criadas
      ↓
Vencimentos definidos
      ↓
Pagamentos registrados
      ↓
Status das parcelas atualizado
      ↓
Saldo da dívida atualizado
```

Caso a dívida esteja associada a uma fatura, o sistema também mantém a referência ao cartão e à fatura correspondente.

---

# Exemplo Completo: Pagamento de Fatura

```text
Pagamento registrado
      ↓
Fatura identificada
      ↓
Valor pago armazenado
      ↓
Saldo recalculado
      ↓
Status atualizado
```

Exemplo:

```text
Valor da Fatura = R$ 551,40
Pagamento = R$ 300,00
Saldo = R$ 251,40
Status = Parcial
```

---

# Benefícios das Automações

A utilização do Google Apps Script trouxe benefícios como:

- redução de tarefas manuais;
- maior padronização;
- criação automática de registros;
- aplicação consistente das regras de negócio;
- melhoria da rastreabilidade;
- redução de erros;
- facilidade de manutenção;
- integração entre coleta e análise;
- preservação do histórico.

---

# Limitações da Arquitetura Atual

O Google Sheets e o Google Apps Script atendem bem ao objetivo do projeto pessoal e de portfólio.

Porém, possuem limitações quando comparados a arquiteturas de maior escala.

Entre elas:

- limites de execução;
- concorrência;
- dependência de planilhas;
- escalabilidade limitada;
- controle de transações mais simples;
- maior dificuldade para grandes volumes de dados.

---

# Evolução Futura

Em uma arquitetura mais robusta, o processamento poderia evoluir para:

```text
Aplicação
    ↓
API
    ↓
Banco de Dados
    ↓
Pipeline / ETL
    ↓
Data Warehouse
    ↓
Power BI
```

Nesse cenário, algumas responsabilidades atualmente executadas pelo Apps Script poderiam ser transferidas para:

- APIs;
- serviços backend;
- stored procedures;
- pipelines de dados;
- ferramentas de orquestração.

Apesar da mudança de tecnologia, as regras de negócio permaneceriam conceitualmente semelhantes.

---

# Segurança e Privacidade

O código de automação pode conter referências privadas utilizadas no ambiente real.

Por esse motivo, antes da publicação no GitHub devem ser removidos ou substituídos:

- IDs de planilhas;
- IDs de formulários;
- URLs privadas;
- endereços de e-mail;
- tokens;
- chaves;
- credenciais;
- dados pessoais;
- logs privados.

A versão pública deve utilizar placeholders quando necessário.

Exemplo:

```javascript
const SPREADSHEET_ID = "SEU_SPREADSHEET_ID";
```

em vez de utilizar um identificador real.

---

# Publicação do Código-Fonte

O código completo de Apps Script não deve ser publicado diretamente sem revisão.

Antes de adicioná-lo ao diretório:

```text
src/apps-script/
```

deve ser realizada uma auditoria para:

1. identificar dados sensíveis;
2. remover informações privadas;
3. substituir identificadores por placeholders;
4. revisar logs;
5. revisar URLs;
6. revisar configurações;
7. validar se o código continua compreensível fora do ambiente pessoal.

---

# Estrutura Planejada para o Código Público

Após a sanitização, a estrutura poderá ser:

```text
src/
└── apps-script/
    ├── 00_Config.gs
    ├── 01_Utils.gs
    ├── 02_Logs.gs
    ├── 03_Validacoes.gs
    ├── 04_ListasApoio.gs
    ├── 05_Faturas.gs
    ├── 06_Dividas.gs
    ├── 07_Pagamentos.gs
    ├── 08_Forms.gs
    ├── 09_Manutencao.gs
    ├── 10_Triggers.gs
    └── 11_Formatacao.gs
```

---

# Resumo Técnico

O Google Apps Script funciona como camada de processamento e automação do projeto.

Sua principal responsabilidade é transformar registros de entrada em dados estruturados e consistentes.

Entre os principais desafios tratados estão:

- múltiplos tipos de movimentação;
- criação automática de parcelas;
- associação entre dívidas, cartões e faturas;
- pagamentos parciais;
- atualização de status;
- tratamento de cartão emprestado;
- rastreabilidade;
- manutenção;
- tratamento de erros;
- reprocessamento.

O Apps Script complementa a arquitetura ao conectar a entrada operacional dos dados com a estrutura analítica utilizada posteriormente no Power BI.
