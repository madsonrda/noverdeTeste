# Data Engineering Technical Test

A Noverde sendo uma empresa de empréstimos, há necessidade de construir modelos de behavior para entender melhor os nossos clientes.
O time de data engineering prepara os dados para a equipe de data science.
O objetivo do exercício é transformar os dados de entrada (raw data) em uma tabela de dados agregados, chamada `loan_documents`.

## Raw data (input)

Esses arquivos contêm dados parecidos com o que extraímos do nosso sistema:
- https://noverde-data-engineering-test.s3.amazonaws.com/loans_sample.csv
- https://noverde-data-engineering-test.s3.amazonaws.com/installments_sample.json
- https://noverde-data-engineering-test.s3.amazonaws.com/payments_sample.parquet

## Aggregated data (output)

O objetivo é gerar a tabela `loan_documents`, no formato parquet. Cada entry (row) da `loan_documents` deve conter os dados relativos a um emprestimo (loan), saber as parcelas (installments), pagamentos (payments), e algumas métricas pre-calculadas. As informações estão armazenadas usando tipos complexos (array, dictionary, nested structures). O arquivo parquet deverá ser compatível com o schema seguinte, para futuras consultas com o Hive Metastore.

```sql
CREATE EXTERNAL TABLE loan_documents (
  loan_id INT,
  period INT,
  accepted_at TIMESTAMP,
  payday INT,
  interest_rate DOUBLE,
  installments MAP<INT, STRING>,
  payments ARRAY<STRUCT<id: INT, payment_date: STRING, method: STRING, amount: DOUBLE>>,
  metrics STRUCT<latency: ARRAY<BOOLEAN>, over30: ARRAY<BOOLEAN>>
)
ROW FORMAT SERDE                                                   
  'org.apache.hadoop.hive.ql.io.parquet.serde.ParquetHiveSerDe'    
WITH SERDEPROPERTIES (                                             
  'path'='<PAQUET FILE PATH>')    
STORED AS INPUTFORMAT                                              
  'org.apache.hadoop.hive.ql.io.parquet.MapredParquetInputFormat'  
OUTPUTFORMAT                                                       
  'org.apache.hadoop.hive.ql.io.parquet.MapredParquetOutputFormat' 
LOCATION '<PAQUET FILE PATH>'
```

NOTA 1: "installments" deve ser um dicionário de (installment_number => due_date)  
NOTE 2: '\<PAQUET FILE PATH\>' é o caminho do arquivo parquet

Linha de exemplo (representada como um doc yaml):

```yaml
loan_id: 9243
period: 3
accepted_at: "2017-05-19 10:09:47.285105"
payday: 4
interest_rate: 3.12
installments:
  1: "2017-06-04"
  2: "2017-07-04"
  3: "2017-08-04"
payments:
- id: "7abaf860-9632-40e9-bfe0-13b67ded8c6f"
  payment_date: "2017-06-06"
  method: boleto
  amount: 283.09
- id: "6abaf860-9632-40e9-bfe0-33b67ded8c6d"
  payment_date: "2017-07-01"
  method: boleto
  amount: 280.00
metrics:
  latency: [false, false, false, true, true, true, false, ...]
  over30: [false, false, false, false, false, false, false, ...]
```

Obs: esses são valores fake

### Métricas (metrics)
Cada métrica está armazenada na forma de um array, cujo index é o número de dias desde a originação (loan.accepted_at)

- metric[0] é o valor da métrica no dia da originação
- metric[1] é o valor da métrica um dia depois da originação

O último index corresponde à data de ontem.

### Definição da métrica "latency"

A métrica `latency` (atraso) é uma lista de boolean, e indica se há atraso no pagamento de alguma parcela, numa determinada data.

### Definição da  métrica "over(n)"

On a given day, the loan has one of the three first payments overdue--"overdue" means that no payment has been received--. The number (n) indicates that the appraisal period is _n_ days after base date.

Por exemplo:

- Se a data de vencimento de uma installment é 2017-06-04, e não foi realizado nenhum pagamento dentro de 30 dias, o valor de Over30 será TRUE depois de 2017-07-04.
- Uma vez que o pagamento foi recebido, o valor de over30 é FALSE

## Requisitos

Você deve utilizar python, pyspark e jupyter. O notebook poderá ser executado da seguinte forma:

```sh
jupyter nbconvert --to notebook --execute script.ipynb
```

Reference: https://nbconvert.readthedocs.io/en/latest/execute_api.html

### Conteúdo do notebook "script.ipynb"

Além de transformar dados, nossos notebooks contém validações básicas, para ajudar nas investigações caso tenha suspeita de erros no dados.

Uma vez que a tabela `loan_documentos` foi gerada, queremos saber a seguinte informação:

Para cada mês de 2019, qual foi a proporção do montante recebido, em relação ao que era esperado? Ex: Vamos supor que em janeiro 2019 recebemos R$ 9000, e o valor total devido neste mês era R$ 10000, você deve criar uma query que retorne da seguinte forma:

|month|year|amount|ratio|
|-|-|-|-|
|01|2019|10000.00|90%|
|02|2019|20000.00|80%|
|03|2019|50000.00|70%|
|04|2019|10000.00|90%|
|05|2019|10000.00|95%|

O resultado pode ser representado por um gráfico da sua escolha, e aparecer dentro do notebook, usando as bibliotecas que ​quiser.

## Entrega

Você deve enviar sua solução em um repositório git (bibucket ou github), com os seguintes arquivos:

- README.md (explicando como fazer o setup e rodar o script)
- script.ipynb (um notebook jupyter contendo o código pyspark e explicacões em markdown)
- diretórios com dados de input e output (organizados de uma maneira apropriada)
- quaisquer outros arquivos que podem vir a ser úteis