### Case iFood
# NYC Yellow Taxi Data - PySpark ETL
**Este projeto realiza a extração, normalização e transformação de dados de corridas de táxi de Nova York (Yellow Taxi), utilizando PySpark e arquivos armazenados em um bucket Amazon S3.**


![Fluxo da ETL](/Workspace/Users/leonardocapriglione@gmail.com/ifood-case/Case-iFood/fluxo.png)


### ⚙️ Pipeline de Processamento - SRC
### 
**1. read_files_by_dates_s3_uc_select_columns:**

Lê os arquivos do bucket S3 com base em uma lista de datas (YYYYMM), e orquestra o pipeline completo:
- Lê múltiplos arquivos Parquet no formato `yellow_tripdata_YYYY-MM.parquet`.
- Chama a função `normalize_dataframe_columns` para padronizar os nomes das colunas.
- Adiciona a coluna `source_file` com o mês de origem de cada arquivo.
- Une todos os DataFrames.
- Chama transform_data para enriquecer e limpar os dados.

**2. normalize_dataframe_columns**

Padroniza os nomes das colunas para evitar erros posteriores e garantir consistência:

- Remove acentos e diacríticos.
- Remove caracteres especiais (mantendo apenas letras, números e underscore).
- Converte tudo para letras minúsculas.

**3. transform_data:**

Enriquece e limpa os dados brutos do Yellow Taxi, adicionando colunas mais descritivas e removendo colunas técnicas:

**Traduz códigos das colunas:**

- VendorID → vendor_name
- RatecodeID → rate_code_name
- store_and_fwd_flag → store_and_fwd_desc
- payment_type → payment_type_desc

Extrai componentes de data e hora das colunas:

`pickup_date, pickup_time, dropoff_date, dropoff_time`

Remove colunas originais: `tpep_pickup_datetime, tpep_dropoff_datetime`

**4. filter_by_pickup_datetime_range:**

Limpa a base com dados fora do range necessário para a análise. 2023-01-01 à 2023-05-31.


**Parâmetros:**

- bucket_name (str): Nome do bucket no S3.
- dates (list[str]): Lista de datas no formato YYYYMM.
- file_format (str): Formato dos arquivos (parquet por padrão).

**Retorno:**

Um único DataFrame contendo todos os dados normalizados e transformados.

**Exemplo de uso:**

Declarar variáveis globais:

imports necessários:

` import os`
` import re`
` from pyspark.sql`
`import SparkSession, DataFrame`
` from pyspark.sql.functions import lit, input_file_name, when, col, to_timestamp, date_format`
`from pyspark.sql.types import StructType, StructField, StringType, DoubleType, IntegerType, LongType`
`from functools import reduce`

`datas = ["202301", "202302", "202303","202304","202305"]` -- Datas necessárias para análise.

`bucket = "landing-layer-ifood"` -- bucket que irá ler os arquivos.

Execução função `main()`

🛠️ Requisitos

Acesso à AWS S3 com permissões de leitura

Configuração do Spark para uso com s3a:// (ex: via Databricks, EMR ou local com Hadoop configurado)

📌 Observações

A função normalize_dataframe_columns é essencial para tratar inconsistências entre arquivos mensais.

A coluna source_file permite rastrear o mês de origem de cada linha do DataFrame final.



### ⚙️ Pipeline de Análise - Analysis
### 

**Este projeto realiza análises de dados de corridas de táxis da frota Yellow Taxi, utilizando PySpark para processar os dados e executar consultas SQL.**

### Objetivos

- Calcular a média do valor total (total_amount) das corridas por mês, filtrando corridas encerradas (com tpep_dropoff_datetime preenchido).

- Calcular a média do número de passageiros (passenger_count) por hora do dia para o mês de maio.

**Descrição do Código**

**1. Preparação dos Dados**

Os dados brutos estão carregados em um DataFrame PySpark chamado df no notebook `src`

O DataFrame é registrado como uma view temporária chamada "yellow_taxi_table" para permitir consultas SQL.

**2. Consulta Média Mensal do Valor Total**

Consulta SQL para calcular a média do valor total `(total_amount)` agrupada por mês (ano e mês no formato yyyy-MM), considerando apenas corridas que possuem `tpep_dropoff_datetime` preenchido para retirar corridas que não foram finalizadas.

**3. Consulta Média de Passageiros por Hora (Mês de Maio)**

Consulta SQL para calcular a média do número de passageiros (passenger_count) por hora do dia, filtrando apenas corridas que ocorreram no mês de **maio de 2023**.

🛠️ **Como Executar**

- Carregue seus dados em um DataFrame PySpark chamado df.

- Execute o script completo para registrar a view e realizar as consultas.

- As mensagens serão impressas no console com os resultados.

**Requisitos**

- Apache Spark com PySpark configurado
- Dados de corridas Yellow Taxi com as colunas mencionadas (`pickup_date`, `pickup_time`, `total_amount`, `passenger_count`, `tpep_dropoff_datetime`)