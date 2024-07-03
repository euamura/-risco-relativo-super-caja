# Super Caja

## Links de apresentação:

- [https://lookerstudio.google.com/reporting/173aef6d-66e5-4f47-b7a5-673459f34794](https://lookerstudio.google.com/reporting/173aef6d-66e5-4f47-b7a5-673459f34794)
- [https://www.loom.com/share/098dd33d3aa6404f89713e53288cc549?sid=02c68f09-3036-4234-b414-eebd74e88df8](https://www.loom.com/share/e67d421dbe4c4129ac57a2d93ec368c4)

- Introdução
    
    O risco relativo é uma medida estatística utilizada em epidemiologia e em diversas outras áreas para avaliar a associação entre a exposição a um fator de risco específico e a ocorrência de um resultado específico, como uma doença ou evento adverso. É calculado como a proporção da incidência do referido desfecho no grupo exposto ao fator de risco em comparação com a incidência no grupo não exposto.
    
    Por outras palavras, o risco relativo fornece uma indicação da probabilidade de ocorrência de um resultado no grupo exposto em comparação com o grupo não exposto. Um risco relativo igual a 1 sugere que não há diferença na incidência entre os dois grupos, enquanto um risco relativo maior que 1 indica um risco maior no grupo exposto, e um risco relativo menor que 1 indica um risco menor no grupo exposto.
    
- Case
    
    Num contexto recente, a descida das taxas de juro no mercado desencadeou um aumento notável na demanda por pedidos de crédito. Os clientes viram este movimento do mercado, como uma oportunidade favorável para financiar grandes compras ou consolidar dívidas existentes, o que elevou o fluxo de pedidos de empréstimo no banco “Super Caja”. A equipe de análise de crédito do banco enfrenta um fardo esmagador devido à análise manual necessária para cada solicitação de empréstimo de clientes individuais. Esta metodologia manual resultou num processo ineficiente e atrasado, que afetou negativamente a eficiência e a velocidade com que os pedidos de empréstimo são processados. A situação tornou-se mais crítica devido à preocupação crescente com a taxa de inadimplência, um problema que afeta cada vez mais o setor financeiro, e aumenta a pressão sobre os bancos para identificar e mitigar riscos associados ao crédito.
    
    Para enfrentar esse desafio, a proposta do banco é a automação do processo de análise de crédito usando técnicas de análise avançadas de dados, com o objetivo de melhorar a eficiência, precisão e rapidez na avaliação de pedidos de crédito. Além disso, o banco já tem uma métrica para identificar clientes com pagamento atrasado, o que poderia ser uma ferramenta valiosa para integrar na classificação de risco no novo sistema automatizado.
    
    O objetivo da análise é identificar o perfil de clientes com risco de inadimplência, montar uma pontuação de crédito a través da análise de dados e avaliar o risco relativo, possibilitando assim, classificar os clientes e futuros clientes em diferentes categorias de risco com base na sua probabilidade de inadimplência. Esta classificação permitirá ao banco tomar decisões informadas sobre a quem conceder crédito, reduzindo assim o risco de empréstimos não reembolsáveis. Além disso, a integração destas métricas fortalecerá a capacidade do modelo de identificar riscos, contribuindo para a solidez financeira e a eficiência operacional do Banco.
    
- Ferramentas
    - Google Colab
    - Apresentações Google
    - Google Looker Studi.
- Linguagens
    
    SQL no BigQuery e Python no Google Colab
    
- Insumos
    
    Dataset: https://drive.google.com/file/d/bc1qf92drq0wwm8w7rnw9d8p4wjvaut2csdd5sg2cx/view
    
    ![glossario](https://github.com/euamura/-risco-relativo-super-caja/assets/156445659/627a62b3-957e-435f-aa2e-6d6f8c5b5736)

    
- Links de referência
    
    [https://docs.google.com/document/d/1q6UPnF3SMgHFcuAsy5DrRsiHGHBxb0qA1aut1Y9daEE/edit](https://docs.google.com/document/d/1q6UPnF3SMgHFcuAsy5DrRsiHGHBxb0qA1aut1Y9daEE/edit)
    

## **1. Processar e preparar a base de dados**

### 1.1. Conectar/importar dados para ferramentas

1.1.1- Baixar insumos, descompactar, importar arquivos no BigQuery e criar 4 tabelas. Nome do projeto no BigQuery: risco-relativo-super-caja; Nome da pasta de conjuto de dados: super_caja; Nome das tabelas: **default, loans_detail, loans_outstanding, user_info.**

### 1.2. Identificar e tratar valores nulos

1.2.1- Identificar e tratar valores nulos usando SQL:

```sql
--consulta de nulos

---planilha default
SELECT *
FROM `super_caja.default`
WHERE user_id IS NULL AND default_flag IS NULL;
----resultado da consulta: não há nulos na planilha default

---planilha loans_detail
SELECT * 
FROM `super_caja.loans_detail`
WHERE user_id IS NULL
OR more_90_days_overdue IS NULL
OR using_lines_not_secured_personal_assets IS NULL
OR number_times_delayed_payment_loan_30_59_days IS NULL
OR debt_ratio IS NULL
OR number_times_delayed_payment_loan_60_89_days IS NULL;
----resultado da consulta: não há nulos na planilha loans_detail

---planilha loans_outstanding
SELECT * 
FROM `super_caja.loans_outstanding`
WHERE loan_id IS NULL
OR user_id IS NULL
OR loan_type IS NULL;
----resultado da consulta: não há nulos na planilha loans_outstanding

---planilha user_info
SELECT * 
FROM `super_caja.user_info`
WHERE user_id IS NULL
OR age IS NULL
OR sex IS NULL
OR last_month_salary IS NULL
OR number_dependents IS NULL
----resultado da consulta: nas colunas last_month_salary e number_dependents há nulos. obs: os nulos de number_dependents são considerados que a pessoa não tem dependentes.

---contagem de nulos planilha user_info
SELECT
COUNT(*) AS nulls_count
FROM `super_caja.user_info`
WHERE last_month_salary IS NULL;
----resultado da consulta: há 7.199 nulos
```

1.2.2- Decisão sobre os dados nulos: Como corresponde a uma boa quantidade do dataset, não excluir. Manter dados correlacionando com os da tabela default, onde existe a variável default_flag para entender se esses registros estão em maior proporção para clientes com flag 1 (mau pagador). Já que o objetivo é encontrar o perfil dos clientes que pagam mal para gerar um mecanismo de regras de aprovação de crédito.

1.2.3- Atualização sobre nulos, após unir tabelas, percebe-se que a planilha user_info tem 425 ids a mais que a tabela loans_detail. Além de alguns nulos na variável number_dependents.

```sql
---contando nulls na tabela principal após unir dados.
SELECT
  SUM(CASE WHEN user_id IS NULL THEN 1 ELSE 0 END) AS user_id_nulls,
  SUM(CASE WHEN last_month_salary IS NULL THEN 1 ELSE 0 END) AS last_month_salary_nulls,
  SUM(CASE WHEN number_dependents IS NULL THEN 1 ELSE 0 END) AS number_dependents_nulls,
  SUM(CASE WHEN loan_type IS NULL THEN 1 ELSE 0 END) AS loan_type_nulls,
  SUM(CASE WHEN loan_id IS NULL THEN 1 ELSE 0 END) AS loan_id_nulls
FROM `super_caja.tabela_principal`;
----resultado da consulta: 0 user_id_nulls | 7.199 last_month_salary_nulls | 943 number_dependents_nulls | 425 loan_type_nulls e loan_id_nulls

```

1.2.4- Tratando nulos em demais variáveis: Usando uma tabela com todas as informações, foi só atualizada as variáveis number_dependents e last_month_salary, onde há nulo, foi atribuído valor 0 usando COALESCE: 

```sql
CREATE OR REPLACE TABLE `super_caja.super_caja_tabela_analise_completa` AS(
  SELECT 
    t.* EXCEPT (last_month_salary , number_dependents),
    COALESCE(number_dependents,0) AS number_dependents,
    COALESCE(last_month_salary,0) AS last_month_salary
  FROM `super_caja.super_caja_tabela_analise_completa` t
);
```

1.2.5- Observações importantes: Conclui-se que há 425 nulos em loan_id, e ao correlacionar com a coluna default_flag, percebe-se que há 61 user_id com default_flag = 1 que têm a linha da coluna loan_id e consequentemente loan_type como null:

```sql
SELECT
  t.user_id,
  t.loan_id,
  t.default_flag
FROM `super_caja.super_caja_tabela_analise_completa` AS t
WHERE loan_id IS NULL AND  default_flag = 1;
```

### 1.3.  Identificar e tratar valores duplicados

1.3.1- Identificar e tratar dados duplicados usando comandos SQL:

```sql
--identificar e tratar duplicados

---planilha default
----criar tabela temporaria com contagem para identificar duplicados
WITH tabela_temp AS (
  SELECT
    *,
    ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY user_id) AS row_num_default
  FROM `super_caja.default`
)
SELECT
  *
FROM
  tabela_temp
WHERE
  row_num_default = 1; 
----após confirmar a tabela_temp, usá-la para atualizar a planilha em questão removendo as duplicadas
  CREATE OR REPLACE TABLE `super_caja.default` AS
WITH tabela_temp AS (
  SELECT
    *,
    ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY user_id) AS row_num_default
  FROM `super_caja.default`
)
SELECT
  *
FROM
  tabela_temp
WHERE
  row_num_default = 1;

---planilha loans detail
----criar tabela temporaria com contagem para identificar duplicados
WITH tabela_temp AS (
  SELECT
    *,
    ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY user_id) AS row_num_loans_detail
  FROM `super_caja.loans_detail`
)
SELECT
  *
FROM
  tabela_temp
WHERE
  row_num = 1;
----após confirmar a tabela_temp, usá-la para atualizar a planilha em questão removendo as duplicadas
  CREATE OR REPLACE TABLE `super_caja.loans_detail` AS
WITH tabela_temp AS (
  SELECT
    *,
    ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY user_id) AS row_numrow_num_loans_detail
  FROM `super_caja.loans_detail`
)
SELECT
  *
FROM
  tabela_temp
WHERE
  row_num_loans_detail = 1;

---planilha loans outstanding
----criar tabela temporaria com contagem para identificar duplicados
WITH tabela_temp AS (
  SELECT
    *,
    ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY user_id) AS row_num_loans_outstanding
  FROM `super_caja.loans_outstanding`
)
SELECT
  *
FROM
  tabela_temp
WHERE
  row_num_loans_outstanding = 1;
----após confirmar a tabela_temp, usá-la para atualizar a planilha em questão removendo as duplicadas
  CREATE OR REPLACE TABLE `super_caja.loans_outstanding` AS
WITH tabela_temp AS (
  SELECT
    *,
    ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY user_id) AS row_num_loans_outstanding
  FROM `super_caja.loans_outstanding`
)
SELECT
  *
FROM
  tabela_temp
WHERE
  row_num_loans_outstanding = 1;

---planilha user_info
----criar tabela temporaria com contagem para identificar duplicados
WITH tabela_temp AS (
  SELECT
    *,
    ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY user_id) AS row_num_user_info
  FROM `super_caja.user_info`
)
SELECT
  *
FROM
  tabela_temp
WHERE
  row_num_user_info = 1;
----após confirmar a tabela_temp, usá-la para atualizar a planilha em questão removendo as duplicadas
  CREATE OR REPLACE TABLE `super_caja.user_info` AS
WITH tabela_temp AS (
  SELECT
    *,
    ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY user_id) AS row_num_user_info
  FROM `super_caja.user_info`
)
SELECT
  *
FROM
  tabela_temp
WHERE
  row_num_user_info = 1;

---planilha loan_outstanding
---conferir duplicados da variavel loan_id
----criar tabela temporaria com contagem para identificar duplicados
WITH tabela_temp AS (
  SELECT
    *,
    ROW_NUMBER() OVER (PARTITION BY loan_id ORDER BY loan_id) AS row_num_loan_id
  FROM `super_caja.loans_outstanding`
)
SELECT
  *
FROM
  tabela_temp
WHERE
  row_num_loan_id = 1;
----após confirmar a tabela_temp, usá-la para atualizar a planilha em questão removendo as duplicadas
  CREATE OR REPLACE TABLE `super_caja.loans_outstanding` AS
WITH tabela_temp AS (
  SELECT
    *,
    ROW_NUMBER() OVER (PARTITION BY loan_id ORDER BY loan_id) AS row_num_loan_id
FROM `super_caja.loans_outstanding`
)
SELECT
  * EXCEPT (row_num_loan_id)
FROM
  tabela_temp
WHERE
  row_num_loan_id = 1;

```

### 1.4. Identificar e gerenciar dados fora do escopo de análise

1.4.1- Identificar dados fora do escopo da análise, considerar a seleção de variáveis e usar CORR para entender a correlação entre variáveis.

1.4.2- O dado considerado fora do escopo foi sex.

1.4.3- Calcar correlação:

- Correlação entre `more_90_days_overdue` e `number_times_delayed_payment_loan_30_59_days` e `number_times_delayed_payment_loan_60_89_days`

Atrasos de mais de 90 dias são um indicativo de inadimplência grave, enquanto atrasos de 30 a 89 dias podem indicar problemas financeiros em estágio inicial. Correlacionar essas variáveis pode ajudar a entender se atrasos menores são precursores de atrasos mais graves.

```sql
--- CORR entre variáveis: more_90_overdue e number_times_delayed_payment_loan_30_59_days
SELECT
  STDDEV_POP(more_90_days_overdue) AS desvio_padrao_more_90_days_overdue,
  STDDEV_POP(number_times_delayed_payment_loan_30_59_days) AS   desvio_padrao_number_times_delayed_payment_loan_30_59_days,
  CORR(more_90_days_overdue, number_times_delayed_payment_loan_30_59_days) AS correlacao_90_days_overdue_delayed_30_59_days
FROM 
  `super_caja.loans_detail`;
----resultado da consulta: há correlação de 0.98 entre as variáveis

--- CORR DE PEARSON entre variáveis: more_90_overdue e number_times_delayed_payment_loan_60_89_days
SELECT
  STDDEV_POP(more_90_days_overdue) AS desvio_padrao_more_90_days_overdue,
  STDDEV_POP(number_times_delayed_payment_loan_60_89_days) AS   desvio_padrao_number_times_delayed_payment_loan_60_89_days,
  CORR(more_90_days_overdue, number_times_delayed_payment_loan_60_89_days) AS correlacao_90_days_overdue_delayed_60_89_days
FROM 
  `super_caja.loans_detail`;
----resultado da consulta: há correlação de 0.99 entre as variáveis
```

- Correlação entre `debt_ratio` e `more_90_days_overdue`

A relação entre dívidas e ativos (`debt_ratio`) pode influenciar a capacidade de pagamento de um indivíduo. Uma alta relação dívida/ativo pode indicar maior risco de inadimplência.

```sql
--- CORR entre variáveis: debt_ratio e more_90_days_overdue
SELECT
  STDDEV(debt_ratio) AS desvio_padrao_debt_ratio,
  STDDEV(more_90_days_overdue) AS desvio_padrao_more_90_days_overdue,
  CORR(debt_ratio,more_90_days_overdue) AS corr_debt_ratio_more_90_days_overdue
FROM `super_caja.loans_detail`;
----resultado da consulta: correlação de  -0.008 entre as variáveis
```

- Correlação entre `using_lines_not_secured_personal_assets` e `default_flag`

Empréstimos sem garantia podem ter um risco maior de inadimplência. Analisar essa correlação pode ajudar a identificar se a ausência de garantias está associada a maiores taxas de inadimplência.

```sql
---  CORR DE PEARSON entre variáveis: using_lines_not_secured_personal_assets e default_flag
SELECT
  STDDEV_POP(using_lines_not_secured_personal_assets) AS desvio_padrao_using_lines_not_secured_personal_assets,
  STDDEV_POP(default_flag) AS desvio_padrao_default,
  CORR(using_lines_not_secured_personal_assets, default_flag) AS corr_using_lines_not_secured_personal_assets_
FROM `super_caja.tabela_principal`;
----resultado da consulta: correlação de  -0.0029 entre as variáveis

```

- Correlação entre `number_times_delayed_payment_loan_60_89_days` e `default_flag`

Atrasos de 60 a 89 dias indicam dificuldades financeiras significativas. Correlacionar com o `default_flag` pode revelar a probabilidade de que esses atrasos resultem em inadimplência.

```sql
--- CORR DE PEARSON entre variáveis: number_times_delayed_payment_loan_60_89_days e default_flag
SELECT
  STDDEV_POP(number_times_delayed_payment_loan_60_89_days) AS desvio_padrao_number_times_delayed_payment_loan_60_89_days,
  STDDEV_POP(default_flag) AS desvio_padrao_default_flag,
  CORR(number_times_delayed_payment_loan_60_89_days,default_flag) AS corr_number_times_delayed_payment_loan_60_89_day_default_flag
FROM `super_caja.tabela_principal`
----resultado da consulta: correlação de  0.27 entre as variáveis
```

### 1.5. Identificar e tratar dados discrepantes em variáveis categóricas

1.5.1- Calcular frequência por variável categórica:

```sql
--Identificar e tratar dados discrepantes em variáveis categóricas
---Contagem de frequencias: default_flag
SELECT default_flag, COUNT(*) AS frequencia
FROM `super_caja.tabela_principal`
GROUP BY default_flag
ORDER BY frequencia ASC;
----resultado da consulta: defalt_flag 1 - 622 | defalt_flag 0 - 34953

---Contagem de frequencias: loan_type
SELECT loan_type, COUNT(*) AS frequencia
FROM `super_caja.tabela_principal`
GROUP BY loan_type
ORDER BY frequencia ASC;
----resultado da consulta: foi encontrado em uma das linhas REAL_STATE (em caixa alta).
----alterando linha em caixa alta:
SELECT
loan_id,
  LOWER(loan_type) AS loan_type
FROM `super_caja.tabela_principal`;
----alterando tabela principal
CREATE OR REPLACE TABLE `super_caja.tabela_principal` AS
SELECT 
  t1.* EXCEPT(loan_type),
  t2.* EXCEPT (loan_id)
FROM `super_caja.tabela_principal` AS t1
LEFT JOIN `super_caja.loan_type_tratado` AS t2
ON t1.loan_id = t2.loan_id;
----após atualizar a tabela. Refazer contagem.
----resultado da consulta de contagem: other - 13.119 | real estate - 22.456

---count de loan_type e default_flag
SELECT loan_type, default_flag, COUNT(*) AS frequencia
FROM `super_caja.tabela_principal`
GROUP BY loan_type, default_flag
ORDER BY loan_type, default_flag;
----resultado da consulta: loan_type other sem default_flag - 12.791 | loan_type other com default_flag - 328 | real estate sem default_flag 22.162 | real estate com default_flag 294
```

1.5.2- Interpretação de contagem de frequência: 

Frequências muito desiguais podem indicar problemas. Por exemplo, se quase todos os registros são 'Não' (0) e poucos são 'Sim' (1), ou vice-versa, pode haver um viés ou erro na coleta de dados.

### 1.6. Identificar e tratar dados discrepantes em variáveis numéricas

1.6.1- Identificar e tratar usando SQL:

```sql
--Identificar e tratar dados discrepantes em variáveis numéricas
---selecionando dados não nulos tabela user_info
SELECT 
  last_month_salary
FROM `super_caja.user_info`
WHERE last_month_salary IS NOT NULL;

---selecionando user_id com valores discrepantes tabela user_info
SELECT * 
FROM   `super_caja.user_info`
WHERE last_month_salary = 1.00E+05
----user_: 4931 | 9466 tem formato de número científico. Ao transformar em inteiro recebem o valor de 100.000

---reconhecendo outliers na variável salario da tabela user_info
SELECT *
FROM`super_caja.user_info`
WHERE last_month_salary > 100000;

```

### 1.7. Verificar e alterar o tipo de dados

1.7.1- Alterar formato e nomes de variáveis:

```sql
--Verificar e alterar o tipo de dados

-- alterando tipo de dados planilha user_info
CREATE OR REPLACE TABLE `super_caja.user_info` AS
SELECT 
  t1.* EXCEPT(last_month_salary, last_month_salary_int),
  CAST(last_month_salary AS INT64) AS last_month_salary
FROM `super_caja.user_info` t1;

--convertendo nomes da variável loan_type com lower
CREATE OR REPLACE TABLE `super_caja.loans_outstanding` AS
SELECT
  loan_id,
  user_id,
  CASE
    WHEN LOWER(loan_type) IN ('real estate', 'real estate') THEN 'real estate'
    WHEN LOWER(loan_type) IN ('other', 'other', 'others') THEN 'other'
    ELSE loan_type
  END AS loan_type_corrigido
FROM
`super_caja.loans_outstanding`;
```

### 1.8. Criar novas variáveis I

1.8.1- Criar variáveis com SQL:

```sql
--criar novas variáveis

--contagem de loan_type por categoria
SELECT
  loan_type_corrigido,
  COUNT(loan_type_corrigido) num_loans_category
FROM `super_caja.loans_outstanding`
GROUP BY loan_type_corrigido;

---criar variaveis: total_loan(mostrar soma de loans) | num_real_estate (quantidade de loans de imóveis) | num_outher (soma de outros emprestimos)
SELECT
  user_id,
  COUNT(DISTINCT loan_id) AS total_loan,
  SUM(CASE WHEN loan_type_corrigido = 'real estate' THEN 1 ELSE 0 END) AS num_real_estate,
  SUM (CASE WHEN loan_type_corrigido = 'other' THEN 1 ELSE 0 END) AS num_other,
FROM `super_caja.loans_outstanding`
GROUP BY user_id;

```

### 1.9. Unir tabelas

1.9.1- Unir tabelas:

```sql
-- unir tabelas
CREATE OR REPLACE TABLE `super_caja.tabela_principal` AS
SELECT
  t1.* EXCEPT(sex),
  t2.loan_id,
  t2.loan_type_corrigido AS loan_type,
  t3.default_flag,
  t4.* EXCEPT(user_id, row_num),
  t5.* EXCEPT(user_id)

FROM `super_caja.user_info` AS t1
LEFT JOIN `super_caja.loans_outstanding` AS t2 ON t1.user_id = t2.user_id
LEFT JOIN `super_caja.default` AS t3 ON t3.user_id = t1.user_id
LEFT JOIN `super_caja.loans_detail` AS t4 ON t4.user_id = t1.user_id
LEFT JOIN `super_caja.loans_count` AS t5 ON t5.user_id = t1.user_id

```

### 1.10. Construir tabelas auxiliares

1.10.1- As  tabelas auxiliares auxiliam na visualização, as criadas aqui, auxiliam na visualização dos nulos e categorias:

```sql
--Construir tabelas auxiliares

---last_month_salary nulls:
SELECT *
FROM `super_caja.tabela_principal`
WHERE last_month_salary IS NULL;
----salvar como view
```

```sql
--Construir tabelas auxiliares

---loan_id nulls:
SELECT * 
FROM `super_caja.tabela_principal`
WHERE loan_id IS NULL;
----salvar como view
```

```sql
--Construir tabelas auxiliares

---default_flag = 1:
SELECT * 
FROM `super_caja.tabela_principal`
WHERE default_flag = 1;
----salvar como view
```

## **2. Fazer uma análise exploratória I**

### 2.1. Calcular quartis, decis ou percentis

2.1.1- Usar SQL para calcular percentis usando uma CTE:

```sql
-- selecionar variaveis importantes
WITH ntile_vars AS (
  SELECT
    user_id,
    default_flag,
    debt_ratio,
    last_month_salary,
    age,
    using_lines_not_secured_personal_assets,
    more_90_days_overdue,
    total_loan,
    -- criar quartil para variaveis importantes 
    NTILE(4) OVER (ORDER BY debt_ratio) AS debt_ratio_ntile,
    NTILE(4) OVER (ORDER BY last_month_salary) AS last_month_salary_ntile,
    NTILE(4) OVER (ORDER BY age) AS age_ntile,
    NTILE(4) OVER (ORDER BY using_lines_not_secured_personal_assets) AS using_lines_ntile,
    NTILE(4) OVER (ORDER BY more_90_days_overdue) AS more_90_days_overdue_ntile,
    NTILE(4) OVER (ORDER BY total_loan) AS total_loan_ntile
  FROM `super_caja.super_caja_tabela_analise_completa`
  
  SELECT * FROM ntile_vars

```

### 2.2. Calcular correlação entre variáveis numéricas

```sql
--Calcular correlaçaõ de pearson de variáveis numéricas

---corr entre: more_90_days_overdue e default_flag
SELECT(
  CORR(default_flag,more_90_days_overdue)
)
FROM `super_caja.tabela_principal`;
----resultado corr: 0.30

---corr entre: number_times_delayed_payment_loan_30_59_days e default_flag
SELECT(
  CORR(t1.default_flag, t1.number_times_delayed_payment_loan_30_59_days)
)
FROM `super_caja.tabela_principal` AS t1;
----resultado corr: 0.29

---corr entre: number_times_delayed_payment_loan_60_89_days e default flag
SELECT(
    CORR(t1.default_flag, t1.number_times_delayed_payment_loan_60_89_days)
)
FROM `super_caja.tabela_principal` AS t1;
----resultado corr: 0.27

---corr entre last_month_salary e default_flag
SELECT
  CORR(last_month_salary, default_flag)
FROM `super_caja.tabela_principal` AS t1;
----resultado corr: -0.02

---corr entre default_flag e total_loan
SELECT
  CORR(default_flag, total_loan)
FROM `super_caja.tabela_principal` AS t1;
----resultado corr: -0.04

---corr entre last_month_salary e total_loan
SELECT
  CORR(last_month_salary, total_loan)
FROM `super_caja.tabela_principal` AS t1;
----resultado corr: -0.10

```

## 3. **Aplicar técnica de análise**

### 3.1. Calcular o risco relativo

3.1.1- Passos para Calcular o Risco Relativo:

1. **Defina os Grupos**: Determine quais grupos você quer comparar. Por exemplo, clientes com alta relação dívida/ativo vs. clientes com baixa relação dívida/ativo.
2. **Calcule as Taxas de Default para Cada Grupo**: Conte quantos clientes de cada grupo são inadimplentes e quantos não são.
3. **Calcule o Risco Relativo**: O risco relativo é a razão entre a taxa de inadimplência do grupo de risco e a taxa de inadimplência do grupo de referência.

3.1.2- Ao criar o quartil e após isso, o decil, foi usado um ordenamento de variáveis com o critério de maior quantidade de RR > 1.

- using_lines_not_secured_personal_assets: 674
- more_90_days_overdue: 643
- age: 512
- last_month_salary: 439
- debt_ratio: 381

3.1.3- Interpretação Risco Relativo:

- **RR > 1**: O evento é mais provável no grupo comparado ao grupo de referência.
- **RR < 1**: O evento é menos provável no grupo comparado ao grupo de referência.
- **RR = 1**: O risco é o mesmo em ambos os grupos.

3.1.4- Para as demais variáveis, foi optado calcular a partir de divisão de quartis:

```sql
-- RISCO RELATIVO COM QUARTIL

--- *debt_ratio*
-- Calcular quartil
WITH ntile_debt_ratio AS (
  SELECT
    user_id,
    default_flag,
    debt_ratio,
    NTILE(4) OVER (ORDER BY debt_ratio DESC) AS debt_ratio_ntile
  FROM `super_caja.tabela_de_analise_super_caja`
),

-- Unir quartil à tabela
data_with_ntile AS (
  SELECT
    u.*,
    n.debt_ratio_ntile
  FROM
    `super_caja.tabela_de_analise_super_caja` AS u
  LEFT JOIN
    ntile_debt_ratio AS n
    ON
    u.user_id = n.user_id
),

-- Selecionar variáveis para calcular risco relativo
risk_relative AS (
  SELECT
    debt_ratio_ntile, -- coluna de ntile
    COUNT(*) AS total_customers,  -- count de total user_id
    SUM(default_flag) AS total_default, -- soma de inadimplentes 
    AVG(default_flag) AS default_rate -- média de inadimplentes
  FROM
    data_with_ntile
  GROUP BY
    debt_ratio_ntile
)

-- Cálculo do risco relativo
SELECT
  debt_ratio_ntile,
  total_customers,
  total_default,
  default_rate,
  default_rate / (SELECT AVG(default_flag) FROM data_with_ntile) AS risk_relative
FROM
  risk_relative
ORDER BY
  debt_ratio_ntile;
----- resultado da consula: quartil 1 e 2 tem RR > 1
---------------------------------------

--- *last_month_salary*
-- Calcular quartil
WITH ntile_last_month_salary AS (
  SELECT
    user_id,
    default_flag,
    last_month_salary,
    NTILE(4) OVER (ORDER BY last_month_salary) AS last_month_salary_ntile
  FROM `super_caja.tabela_de_analise_super_caja`
),

-- Unir quartil à tabela
data_with_ntile AS (
  SELECT
    u.*,
    n.last_month_salary_ntile
  FROM
    `super_caja.tabela_de_analise_super_caja` AS u
  LEFT JOIN
    ntile_last_month_salary AS n
    ON
    u.user_id = n.user_id
),

-- Selecionar variáveis para calcular risco relativo
risk_relative AS (
  SELECT
    last_month_salary_ntile, -- coluna de ntile
    COUNT(*) AS total_customers,  -- count de total user_id
    SUM(default_flag) AS total_default, -- soma de inadimplentes 
    AVG(default_flag) AS default_rate -- média de inadimplentes
  FROM
    data_with_ntile
  GROUP BY
    last_month_salary_ntile
)
-- Cálculo do risco relativo
SELECT
  last_month_salary_ntile,
  total_customers,
  total_default,
  default_rate,
  default_rate / (SELECT AVG(default_flag) FROM data_with_ntile) AS risk_relative
FROM
  risk_relative
ORDER BY
  last_month_salary_ntile;
----- resultado da consula: quartil 1 e 2 tem RR > 1

-----------------------------------------------

--- *age*
-- Calcular quartil
WITH ntile_age AS (
  SELECT
    user_id,
    default_flag,
    age,
    NTILE(4) OVER (ORDER BY age) AS age_ntile
  FROM `super_caja.tabela_de_analise_super_caja`
),

-- Unir quartil à tabela
data_with_ntile AS (
  SELECT
    u.*,
    n.age_ntile
  FROM
    `super_caja.tabela_de_analise_super_caja` AS u
  LEFT JOIN
    ntile_age AS n
    ON
    u.user_id = n.user_id
),

-- Selecionar variáveis para calcular risco relativo
risk_relative AS (
  SELECT
    age_ntile, -- coluna de ntile
    COUNT(*) AS total_customers,  -- count de total user_id
    SUM(default_flag) AS total_default, -- soma de inadimplentes 
    AVG(default_flag) AS default_rate -- média de inadimplentes
  FROM
    data_with_ntile
  GROUP BY
    age_ntile
)
-- Cálculo do risco relativo
SELECT
  age_ntile,
  total_customers,
  total_default,
  default_rate,
  default_rate / (SELECT AVG(default_flag) FROM data_with_ntile) AS risk_relative
FROM
  risk_relative
ORDER BY
  age_ntile;
----- resultado da consula: quartil 1 e 2 tem RR > 1
--------------------------------------------------

--- *using_lines_not_secured_personal_assets*
-- Calcular quartil
WITH ntile_using_lines_not_secured_personal_assets AS (
  SELECT
    user_id,
    default_flag,
    using_lines_not_secured_personal_assets,
    NTILE(4) OVER (ORDER BY using_lines_not_secured_personal_assets DESC) AS using_lines_not_secured_personal_assets_ntile
  FROM `super_caja.tabela_de_analise_super_caja`
),

-- Unir quartil à tabela
data_with_ntile AS (
  SELECT
    u.*,
    n.using_lines_not_secured_personal_assets_ntile
  FROM
    `super_caja.tabela_de_analise_super_caja` AS u
  LEFT JOIN
    ntile_using_lines_not_secured_personal_assets AS n
    ON
    u.user_id = n.user_id
),

-- Selecionar variáveis para calcular risco relativo
risk_relative AS (
  SELECT
    using_lines_not_secured_personal_assets_ntile, -- coluna de ntile
    COUNT(*) AS total_customers,  -- count de total user_id
    SUM(default_flag) AS total_default, -- soma de inadimplentes 
    AVG(default_flag) AS default_rate -- média de inadimplentes
  FROM
    data_with_ntile
  GROUP BY
    using_lines_not_secured_personal_assets_ntile
)
-- Cálculo do risco relativo
SELECT
  using_lines_not_secured_personal_assets_ntile,
  total_customers,
  total_default,
  default_rate,
  default_rate / (SELECT AVG(default_flag) FROM data_with_ntile) AS risk_relative
FROM
  risk_relative
ORDER BY
  using_lines_not_secured_personal_assets_ntile;
----- resultado da consula: quartil 1 tem RR > 1

-------------------------------------------------

--- *total_loan*
-- Calcular quartil
WITH ntile_total_loan AS (
  SELECT
    user_id,
    default_flag,
    total_loan,
    NTILE(4) OVER (ORDER BY total_loan) AS total_loan_ntile
  FROM `super_caja.tabela_de_analise_super_caja`
),

-- Unir quartil à tabela
data_with_ntile AS (
  SELECT
    u.*,
    n.total_loan_ntile
  FROM
    `super_caja.tabela_de_analise_super_caja` AS u
  LEFT JOIN
    ntile_total_loan AS n
    ON
    u.user_id = n.user_id
),

-- Selecionar variáveis para calcular risco relativo
risk_relative AS (
  SELECT
    total_loan_ntile, -- coluna de ntile
    COUNT(*) AS total_customers,  -- count de total user_id
    SUM(default_flag) AS total_default, -- soma de inadimplentes 
    AVG(default_flag) AS default_rate -- média de inadimplentes
  FROM
    data_with_ntile
  GROUP BY
    total_loan_ntile
)
-- Cálculo do risco relativo
SELECT
  total_loan_ntile,
  total_customers,
  total_default,
  default_rate,
  default_rate / (SELECT AVG(default_flag) FROM data_with_ntile) AS risk_relative
FROM
  risk_relative
ORDER BY
  total_loan_ntile;
----- resultado da consula: quartil 1 tem RR > 1

-------------------------------------------------
--- *more_90_days_overdue*
-- Calcular quartil
WITH ntile_more_90_days_overdue AS (
  SELECT
    user_id,
    default_flag,
    more_90_days_overdue,
    NTILE(4) OVER (ORDER BY more_90_days_overdue DESC) AS more_90_days_overdue_ntile
  FROM `super_caja.tabela_de_analise_super_caja`
),

-- Unir quartil à tabela
data_with_ntile AS (
  SELECT
    u.*,
    n.more_90_days_overdue_ntile
  FROM
    `super_caja.tabela_de_analise_super_caja` AS u
  LEFT JOIN
    ntile_more_90_days_overdue AS n
    ON
    u.user_id = n.user_id
),

-- Selecionar variáveis para calcular risco relativo
risk_relative AS (
  SELECT
    more_90_days_overdue_ntile, -- coluna de ntile
    COUNT(*) AS total_customers,  -- count de total user_id
    SUM(default_flag) AS total_default, -- soma de inadimplentes 
    AVG(default_flag) AS default_rate -- média de inadimplentes
  FROM
    data_with_ntile
  GROUP BY
    more_90_days_overdue_ntile
)
-- Cálculo do risco relativo
SELECT
  more_90_days_overdue_ntile,
  total_customers,
  total_default,
  default_rate,
  default_rate / (SELECT AVG(default_flag) FROM data_with_ntile) AS risk_relative
FROM
  risk_relative
ORDER BY
  more_90_days_overdue_ntile;
----- resultado da consula: quartil 1 tem RR > 1

---------------------------------------

--- *quartil ordenado por varias variaveis ordenadas pela quantidade de defaults com RR superior a 1*
--- *using_lines_not_secured_personal_assets, more_90_days_overdue, age, last_month_salary, debt_ratio*
-- Calcular quartil
WITH ntile_geral AS (
  SELECT
    user_id,
    default_flag,
    using_lines_not_secured_personal_assets,
    more_90_days_overdue,
    age,
    last_month_salary,
    debt_ratio,
    NTILE(4) OVER (ORDER BY using_lines_not_secured_personal_assets DESC, more_90_days_overdue DESC, age, last_month_salary, debt_ratio DESC) AS geral_ntile
  FROM `super_caja.tabela_de_analise_super_caja`
),

-- Unir quartil à tabela
data_with_ntile AS (
  SELECT
    u.*,
    n.geral_ntile
  FROM
    `super_caja.tabela_de_analise_super_caja` AS u
  LEFT JOIN
    ntile_geral AS n
    ON
    u.user_id = n.user_id
),

-- Selecionar variáveis para calcular risco relativo
risk_relative AS (
  SELECT
    geral_ntile, -- coluna de ntile
    COUNT(*) AS total_customers,  -- count de total user_id
    SUM(default_flag) AS total_default, -- soma de inadimplentes 
    AVG(default_flag) AS default_rate -- média de inadimplentes
  FROM
    data_with_ntile
  GROUP BY
    geral_ntile
)
-- Cálculo do risco relativo
SELECT
  geral_ntile,
  total_customers,
  total_default,
  default_rate,
  default_rate / (SELECT AVG(default_flag) FROM data_with_ntile) AS risk_relative
FROM
  risk_relative
ORDER BY
  geral_ntile;
----- resultado da consula: quartil 1 tem RR > 1

-------------------------------------------------

--- *decil ordenado por varias variaveis ordenadas pela quantidade de defaults com RR superior a 1*
--- *using_lines_not_secured_personal_assets, more_90_days_overdue, age, last_month_salary, debt_ratio*
-- Calcular quartil
WITH ntile_geral AS (
  SELECT
    user_id,
    default_flag,
    using_lines_not_secured_personal_assets,
    more_90_days_overdue,
    age,
    last_month_salary,
    debt_ratio,
    NTILE(10) OVER (ORDER BY using_lines_not_secured_personal_assets DESC, more_90_days_overdue DESC, age, last_month_salary, debt_ratio DESC) AS geral_ntile
  FROM `super_caja.tabela_de_analise_super_caja`
),

-- Unir quartil à tabela
data_with_ntile AS (
  SELECT
    u.*,
    n.geral_ntile
  FROM
    `super_caja.tabela_de_analise_super_caja` AS u
  LEFT JOIN
    ntile_geral AS n
    ON
    u.user_id = n.user_id
),

-- Selecionar variáveis para calcular risco relativo
risk_relative AS (
  SELECT
    geral_ntile, -- coluna de ntile
    COUNT(*) AS total_customers,  -- count de total user_id
    SUM(default_flag) AS total_default, -- soma de inadimplentes 
    AVG(default_flag) AS default_rate -- média de inadimplentes
  FROM
    data_with_ntile
  GROUP BY
    geral_ntile
)
-- Cálculo do risco relativo
SELECT
  geral_ntile,
  total_customers,
  total_default,
  default_rate,
  default_rate / (SELECT AVG(default_flag) FROM data_with_ntile) AS risk_relative
FROM
  risk_relative
ORDER BY
  geral_ntile;
----- resultado da consula: decil  1 e 2 tem RR > 1

```

### 3.2. Validar hipótese

3.2.1- Hipótese 1 Alternativa - Os mais jovens correm um risco maior de não pagamento. 

- O resultado do RR sobre esta hipótese é 0.15 que indica que a taxa de inadimplência dos clientes mais velhos (quartil 4) é 15% da taxa de inadimplência dos clientes mais jovens (quartil 1). Isso sugere que os clientes mais jovens têm uma probabilidade de inadimplência 85% maior que os mais velhos.

3.2.2- Hipótese 2 Nula - Pessoas com mais empréstimos ativos correm maior risco de serem maus pagadores.

- O resultado RR sobre esta hipótese é 0.34 que indica que a taxa de inadimplência dos usuários no quartil com maior número de empréstimos (quartil 4) é 34% menor do que a taxa de inadimplência dos clientes no quartil com menor número de empréstimos (quartil 1). Ou seja clientes com menor número de empréstimos (quartil 1) tem 66% de chance de inadimplência com relação ao quartil 4.

3.2.3- Hipótese 3 Alternativa - Pessoas que atrasaram seus pagamentos por mais de 90 dias correm maior risco de serem maus pagadores. 

- O resultado RR sobre esta hipótese é 0.01 que indica que a taxa de inadimplência dos clientes no quartil com menos vezes atraso após 90 dias (quartil 4) é 1% da taxa de inadimplência dos clientes no quartil com mais vezes atraso após 90 dias (quartil 1). Indicando uma forte correlação entre o número de vezes com um número de dias de atraso e a probabilidade de inadimplência. O risco relativo de 0.01 mostra que clientes com menos dias de atraso (quartil 4) são significativamente menos propensos à inadimplência comparados aos clientes com mais vezes com flag de atraso pós 90 dias (quartil 1).

### 3.3. Classificação de clientes

3.3.1 - Primeiramente, é definido os quartis de corte para bom ou mau pagador, partir do risco relativo. Quando o RR é ≥ temos um ‘mau pagador’, se RR < 1 ‘bom pagador’. Após o calculo de RR as variáveis selecionadas como importantes resultaram na seguinte tabela:

**QUARTIS COM RR ≥ 1** 

| age | last_month_salary | using_lines | more_90_days_overdue |
| --- | --- | --- | --- |
| 1 - 2 | 1 - 2 | 4  | 4 |

![tabela-risco-relativo-salario](https://github.com/euamura/-risco-relativo-super-caja/assets/156445659/cc709f46-8c96-4117-a2d2-65e58a335fcc)
![tabela-risco-relativo-uso-de-credito](https://github.com/euamura/-risco-relativo-super-caja/assets/156445659/8c25e631-f53c-4fa3-adbd-03d0574b0b42)
![tabela-risco-relativo-idade](https://github.com/euamura/-risco-relativo-super-caja/assets/156445659/a2abea63-2f0c-41a1-abba-bd550bab04ef)
![tabela-risco-relativo-atraso](https://github.com/euamura/-risco-relativo-super-caja/assets/156445659/c79201df-5564-4248-b626-5e051b00d52e)


3.3.2 - Aplicar segmentação usando CASE WHEN em SQL:

```sql
-- classificando clientes em bom ou mal pagador -- 
SELECT
  t.*,
  CASE WHEN age_ntile <= 2 THEN 'mau pagador' ELSE 'bom pagador' END AS age_class,
  CASE WHEN using_lines_ntile = 4 THEN 'mau pagador' ELSE 'bom pagador' END AS using_lines_class,
  CASE WHEN more_90_days_overdue_ntile = 4 THEN 'mau pagador' ELSE 'bom pagador' END AS overdue_class,
  CASE WHEN last_month_salary_ntile <= 2 THEN 'mau pagador' ELSE 'bom pagador' END AS salary_class
FROM `super_caja.super_caja_tabela_analise_completa` AS t

-- atualizar tabela de analise completa com as novas colunas --
CREATE OR REPLACE TABLE `super_caja.super_caja_tabela_analise_completa` AS
  SELECT
    t.*,
    CASE WHEN age_ntile <= 2 THEN 'mau pagador' ELSE 'bom pagador' END AS age_class,
    CASE WHEN using_lines_ntile = 4 THEN 'mau pagador' ELSE 'bom pagador' END AS using_lines_class,
    CASE WHEN more_90_days_overdue_ntile = 4 THEN 'mau pagador' ELSE 'bom pagador' END AS overdue_class,
    CASE WHEN last_month_salary_ntile <= 2 THEN 'mau pagador' ELSE 'bom pagador' END AS salary_class
  FROM `super_caja.super_caja_tabela_analise_completa` AS t
```

### 3.3. Criar novas variáveis II

3.3.1- Criar tabela com variáveis dummy e score

```sql

--- CRIAR VARIAVEIS DUMMY
-- Criar variáveis dummy para cada classificação
 WITH data_with_dummies AS (
  SELECT
    d.*,
    CASE WHEN age_class = 'mau pagador' THEN 1 ELSE 0 END AS age_dummy,
    CASE WHEN salary_class = 'mau pagador' THEN 1 ELSE 0 END AS salary_dummy,
    CASE WHEN overdue_class = 'mau pagador' THEN 1 ELSE 0 END AS overdue_dummy,
    CASE WHEN using_lines_class = 'mau pagador' THEN 1 ELSE 0 END AS using_line_dummy
  FROM `super_caja.super_caja_tabela_analise_completa` AS d
)
SELECT * FROM data_with_dummies;

-- Atualizar tabela com variaveis dummies
CREATE OR REPLACE TABLE `super_caja.super_caja_tabela_analise_completa` AS
  SELECT
    d.*,
    CASE WHEN age_class = 'mau pagador' THEN 1 ELSE 0 END AS age_dummy,
    CASE WHEN salary_class = 'mau pagador' THEN 1 ELSE 0 END AS salary_dummy,
    CASE WHEN overdue_class = 'mau pagador' THEN 1 ELSE 0 END AS overdue_dummy,
    CASE WHEN using_lines_class = 'mau pagador' THEN 1 ELSE 0 END AS using_line_dummy
  FROM `super_caja.super_caja_tabela_analise_completa` AS d

--------------------------------------------------------
--- CALCULAR SCORE

-- Calcular score a partir das variaveis dummies
WITH score_dummies AS(
  SELECT
    t.*,
    (age_dummy + salary_dummy + overdue_dummy + using_line_dummy) AS score
FROM `super_caja.super_caja_tabela_analise_completa` AS t
)
SELECT * FROM score_dummies

-- Atualizar tabela com a coluna de score
CREATE OR REPLACE TABLE `super_caja.super_caja_tabela_analise_completa` AS
  SELECT
    t.*,
    (age_dummy + salary_dummy + overdue_dummy + using_line_dummy) AS score
  FROM `super_caja.super_caja_tabela_analise_completa` AS t

```

### 3.4. Aplicar segmentação

3.4.1- Segmentar usando SQL, usando distribuição de score para escolher ponto de corte, usando matriz de confusão para avaliar o modelo:

```sql
---- segmentação de clientes

-- analisando distribuição de score
WITH score_distribution AS (
  SELECT
    score,
    COUNT(*) AS total_clients,
    SUM(CASE WHEN default_flag = 1 THEN 1 ELSE 0 END) AS total_defaulters,
    SUM(CASE WHEN default_flag = 1 THEN 1 ELSE 0 END) / COUNT(*) AS default_rate,
  FROM `super_caja.super_caja_tabela_analise_completa`
  GROUP BY score
)
SELECT * FROM score_distribution
ORDER BY score;

-- segmentando os clientes em alto e baixo risco com corte score 4
WITH segmentacao AS(
  SELECT
    t.*,
    CASE WHEN score =4 THEN 1 ELSE 0 END AS segmentacao_dummy
  FROM `super_caja.supercaja_quartile_dummy_score` AS t
),
------SELECT * FROM segmentacao;

classificacao AS(
  SELECT
  *,
  CASE WHEN segmentacao_dummy = 0 THEN 'baixo risco' ELSE 'alto risco' END AS classificacao
  FROM segmentacao
),
------SELECT * FROM classificacao;

-- CTE para calcular a matriz de confusão
matriz_confusao AS (
  SELECT
    COUNT(*) AS total,
    SUM(CASE WHEN default_flag = 1 AND segmentacao_dummy = 1 THEN 1 ELSE 0 END) AS true_positives,
    SUM(CASE WHEN default_flag = 0 AND segmentacao_dummy = 1 THEN 1 ELSE 0 END) AS false_positives,
    SUM(CASE WHEN default_flag = 1 AND segmentacao_dummy = 0 THEN 1 ELSE 0 END) AS false_negatives,
    SUM(CASE WHEN default_flag = 0 AND segmentacao_dummy = 0 THEN 1 ELSE 0 END) AS true_negatives
  FROM classificacao
)

-- Selecionar a matriz de confusão
SELECT
  true_positives,
  false_positives,
  false_negatives,
  true_negatives
FROM matriz_confusao;

```

3.4.2- Atribuindo pontuação positiva aos clientes:

```sql
-- criação de pontuação positiva a partir do score

SELECT 
  t.*,
  CASE 
    WHEN score = 0 THEN 1000 
    WHEN score = 1 THEN 800 
    WHEN score = 2 THEN 600 
    WHEN score = 3 THEN 400 
    ELSE 200 
  END AS pontuacao
FROM 
  `super_caja.super_caja_tabela_analise_completa` AS t

-- atualizando tabela com a nova coluna

CREATE OR REPLACE TABLE `super_caja.super_caja_tabela_analise_completa` AS
  SELECT 
  t.*,
  CASE 
    WHEN score = 0 THEN 1000 
    WHEN score = 1 THEN 800 
    WHEN score = 2 THEN 600 
    WHEN score = 3 THEN 400 
    ELSE 200 
  END AS pontuacao
FROM 
  `super_caja.super_caja_tabela_analise_completa` AS t

```

### 3.5. Regressão logística

3.5.1 - Acessar link do Google Colab.

https://colab.research.google.com/drive/bc1qf92drq0wwm8w7rnw9d8p4wjvaut2csdd5sg2cx-Db2ci?usp=sharing

## 4. Fazer análise exploratória II
![analise-exploratoria-super-caja-1](https://github.com/euamura/-risco-relativo-super-caja/assets/156445659/a59d3abc-9277-4583-b8a4-fb915563c082)
![analise-exploratoria-super-caja-2](https://github.com/euamura/-risco-relativo-super-caja/assets/156445659/4135592b-f7fb-4191-9b62-ea9ca71a0917)
![analise-exploratoria-super-caja-3](https://github.com/euamura/-risco-relativo-super-caja/assets/156445659/58b05d85-94ee-42f4-a2ba-f27c7e70d042)
![analise-exploratoria-super-caja-4](https://github.com/euamura/-risco-relativo-super-caja/assets/156445659/5b8269c4-447c-4424-8dad-8de19c348769)
![analise-exploratoria-super-caja-5](https://github.com/euamura/-risco-relativo-super-caja/assets/156445659/ebe4202f-bd4c-4128-8b4c-bd4c2c162f98)
![analise-exploratoria-super-caja-6](https://github.com/euamura/-risco-relativo-super-caja/assets/156445659/1ab5d33e-cbeb-4825-a59b-73d58688b2fb)
