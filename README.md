#Ingestão de Dados - Yellow Taxis NYC (Jan-Mai 2023)
Solução para ingestão, transformação e análise dos dados de corridas de táxis amarelos de Nova York, utilizando PySpark com Arquitetura Medallion (Bronze / Silver), preparada para execução no Databricks com Unity Catalog.

#Estrutura do Projeto
ingestiondbrickscase/
├── analysis/
│   └── analysis.ipynb          # Notebook Jupyter (.ipynb): mesmo conteúdo, para import direto
├── src/
│   └── ingestor.ipynb          # Notebook Jupyter (.ipynb): mesmo conteúdo, para import direto
├── README.md
Arquitetura Medallion com Unity Catalog
┌──────────────────┐     ┌──────────────────┐     ┌──────────────────┐
│   Landing Zone   │     │     Bronze       │     │     Silver       │
│  (UC Volume)     │────>│ (Managed Table)  │────>│ (Managed Table)  │
│                  │     │                  │     │                  │
│ /Volumes/CATALOG │     │ CATALOG.bronze.  │     │ CATALOG.silver.  │
│ /landing/        │     │ yellow_taxi_trips│     │ yellow_taxi_trips│
│ yellow_taxi/     │     │                  │     │                  │
└──────────────────┘     └──────────────────┘     └──────────────────┘
#Camada	Descrição	Formato	Localização
#Landing Zone (Volume)	Arquivos Parquet no Volume do Unity Catalog	Parquet	/Volumes/{CATALOG}/landing/yellow_taxi
Bronze	Dados brutos + metadados (_data_ingestao, _arquivo_origem)	Delta (Managed Table)	{CATALOG}.bronze.yellow_taxi_trips
Silver	Dados limpos e tipados com 5 colunas obrigatórias	Delta (Managed Table)	{CATALOG}.silver.yellow_taxi_trips
Pipeline de Ingestão (src/ingestion.ipynb)
Etapa 1 — Volume → Bronze:

Lê cada arquivo Parquet da Landing Zone (Volume do Unity Catalog)
Adiciona metadados de ingestão (_data_ingestao, _arquivo_origem, _ano_mes_referencia)
Escreve como tabela gerenciada Delta com saveAsTable()
Tabela: {CATALOG}.bronze.yellow_taxi_trips
Etapa 2 — Bronze → Silver:

Lê dados da camada Bronze via spark.table()
Seleciona e normaliza as 5 colunas obrigatórias:
VendorID (IntegerType)
passenger_count (DoubleType)
total_amount (DoubleType)
tpep_pickup_datetime (TimestampType)
tpep_dropoff_datetime (TimestampType)
Aplica limpeza (remove nulos, valores negativos)
Adiciona coluna ano_mes e particiona por ela
Tabela: {CATALOG}.silver.yellow_taxi_trips
Tecnologias
Databricks com Unity Catalog: Plataforma de execução e governança de dados
PySpark: Processamento distribuído dos dados
Delta Lake: Formato de armazenamento com suporte a ACID (tabelas gerenciadas)
Spark SQL: Consultas analíticas sobre os dados
Unity Catalog Volumes: Armazenamento de arquivos na Landing Zone
Como Executar
Pré-requisitos
Workspace Databricks com Unity Catalog habilitado
Catálogo configurado (padrão: main)
1. Preparar o Unity Catalog
-- Criar os schemas necessários (executado automaticamente pelo notebook)
CREATE SCHEMA IF NOT EXISTS main.landing;
CREATE SCHEMA IF NOT EXISTS main.bronze;
CREATE SCHEMA IF NOT EXISTS main.silver;
2. Upload dos Parquet para o Volume
No Databricks, vá em Catalog → selecione seu catálogo → schema landing
Crie um Volume chamado yellow_taxi
Faça upload dos 5 arquivos .parquet da pasta data/landing_zone/ deste repositório para o Volume
O caminho final dos arquivos será: /Volumes/main/landing/yellow_taxi/yellow_tripdata_2023-XX.parquet

3. Importar Notebooks no Databricks
No Databricks, vá em Workspace → Import
Importe os arquivos .ipynb (recomendado) ou .py:
src/ingestion.ipynb → Notebook de ingestão (Bronze + Silver)
analysis/queries.ipynb → Notebook de análises
Nota: Os arquivos .ipynb podem ser importados diretamente. Os .py no formato Databricks notebook também são suportados.

4. Ajustar o Catálogo (se necessário)
Se seu catálogo não se chama main, edite a variável CATALOG na primeira célula de código dos notebooks:

CATALOG = "seu_catalogo"
5. Executar os Notebooks
Execute primeiro o notebook src/ingestion (cria as camadas Bronze e Silver)
Execute o notebook analysis/queries (executa as análises sobre a Silver)
#Análises Realizadas
Pergunta 1
Qual a média de valor total (total_amount) recebido em um mês considerando todos os yellow táxis da frota?

Implementada em SQL e PySpark — calcula o AVG(total_amount) agrupado por ano_mes.

Pergunta 2
Qual a média de passageiros (passenger_count) por cada hora do dia que pegaram táxi no mês de maio considerando todos os táxis da frota?

Implementada em SQL e PySpark — filtra os dados de maio/2023 e calcula o AVG(passenger_count) agrupado por HOUR(tpep_pickup_datetime).

#Dados de Exemplo
Os arquivos Parquet de exemplo (Jan-Mai 2023) estão disponíveis na pasta data/landing_zone/. Estes são os mesmos dados disponíveis no site do NYC TLC.

#Fonte dos Dados
NYC Taxi & Limousine Commission - Trip Record Data

Período: Janeiro a Maio de 2023 (Yellow Taxi Trip Records)

