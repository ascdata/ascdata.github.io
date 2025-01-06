---
title:  "Data Engineering Zoomcamp: BigQuery"
mathjax: true
layout: post
categories: media
---

The third week of the Data Engineering Zoomcamp focused on BigQuery and Data Warehouse architectures. 
To differentiate between terms, I'd like to briefly explain Data Warehouse, Data Lake, Data Mart, and Data Lakehouse. 
Then I'll describe setting up external tables in BigQuery, as well as partitioning and clustering. After that I'll provide some information about the machine learning model.

# Architectures

**Data Warehouse:**
A Data Warehouse is like a organized digital storage system used for storing and managing large volumes of structured data. 
Examples include Google's BigQuery, Amazon's Redshift, and Azure's SQL Data Warehouse.

**Data Lake:**
A Data Lake is a storage repository where data is stored in its raw form without any specific organization. 
It's like a big pool of data waiting to be explored and analyzed. Examples include Google's Cloud Storage, Amazon's S3 (Simple Storage Service), and Azure Data Lake Storage.

**Data Mart:**
A Data Mart is a subset of a Data Warehouse for the specific needs of a particular business unit or department.

**Data Lakehouse:**
A Data Lakehouse combines the structure of a Data Warehouse with the benefits of a Data Lake. 
It integrates both structured and unstructured data, providing a platform for analytics and data processing.

![Mage Pipeline](https://raw.githubusercontent.com/ascdata/ascdata.github.io/master/_posts/media/week_3_bigquery.jpg)

# Google BigQuery

Google BigQuery is a cloud-based big data analytics web service for processing large data sets. 

It's described as a 
"fully managed, petabyte-scale, and cost-effective analytics data warehouse that lets you run analytics over vast amounts of data in near real time. 
With BigQuery, there's no infrastructure to set up or manage, letting you focus on finding meaningful insights using GoogleSQL and taking advantage of flexible pricing models across on-demand and flat-rate options."

BigQuery is often described as serverless, which really just means for us the end user, there are no servers in which we must manage.

In regards to pricing, BigQuery has two main components:

- Compute pricing: Costs to process queries, including SQL queries, Machine Learning queries, etc.
- Storage pricing: Costs of storing data in BigQuery

Queries are typically billed based on an on-demand pricing structure. This means you're charged only for the amount of data your queries scan. 
The initial terabyte (TiB) per month is provided at no cost, while any additional TiBs are charged at a rate of $6.25, as of the latest pricing information.

# Creating an external table

An External Table is a table that is created using a data source that is not stored in BigQuery storage. 
For my purposes, I will be creating an external table using the taxi data I ingested earlier into my Google Cloud Storage bucket. 
There are other external data sources you can also utilize, such such as a google cloud database or a different cloud product altogether.

```sql
-- Creating external table referring to GCP path
CREATE OR REPLACE EXTERNAL TABLE `dtc-de-zoomcamp-12345.ny_taxi.external_yellow_tripdata`
OPTIONS (
  format = 'parquet',
  uris = ['gs://mage-zoomcamp-bucket/yellow/yellow_tripdata_2019.parquet',
  'gs://mage-zoomcamp-bucket/yellow/yellow_tripdata_2020.parquet']
);
```

# Partitioning and Clustering

In data engineering, it's vital to optimize query performance. 
BigQuery employs two primary strategies—partitioning and clustering—to improve the efficiency of data warehouse queries. 
Below, I'll showcase an example subset of the taxi data before implementing any partitioning or clustering techniques. 

| Row | VendorID | Pickup Datetime      | Trip Distance |
|-----|----------|----------------------|---------------|
| 1   | 2        | 2020-01-14 12:57:37  | 1.2           |
| 2   | 2        | 2020-01-14 17:30:02  | 0.99          |
| 3   | 2        | 2020-01-14 22:33:45  | 0.43          |
| 4   | 2        | 2020-01-14 07:57:20  | 1.8           |
| 5   | 2        | 2020-01-14 13:26:44  | 1.0           |
| 6   | 2        | 2020-01-14 22:46:25  | 0.53          |
| 7   | 2        | 2020-01-14 07:35:59  | 1.39          |

# Partitioning
Partitioning involves dividing a table into segments based on specific column values, often dates. 
This approach is particularly useful for large datasets, as it allows queries to scan only relevant partitions, reducing the amount of data processed and thereby improving performance.

In my project, I've created a partitioned table yellow_tripdata_partitoned, partitioning it by the tpep_pickup_datetime date. This significantly reduced the data scanned in the queries, as demonstrated by the comparison between partitioned and non-partitioned table queries.

Partition for 2020-01-14

| Row | VendorID | Pickup Date | Trip Distance |
|-----|----------|-------------|---------------|
| 1   | 3        | 2020-01-14  | 1.2           |
| 2   | 2        | 2020-01-14  | 0.99          |
| 3   | 1        | 2020-01-14  | 0.43          |

Partition for 2020-01-15

| Row | VendorID | Pickup Date | Trip Distance |
|-----|----------|-------------|---------------|
| 4   | 2        | 2020-01-15  | 1.8           |
| 5   | 2        | 2020-01-15  | 1.0           |

Partition for 2020-01-16

| Row | VendorID | Pickup Date | Trip Distance |
|-----|----------|-------------|---------------|
| 6   | 2        | 2020-01-16  | 0.53          |
| 7   | 2        | 2020-01-16  | 1.39          |

# Clustering
Clustering complements partitioning by organizing data within each partition. It sorts the data based on specified columns, which can further speed up query performance, especially for filter-heavy queries.

I've enhanced my partitioned table by clustering it by VendorID. This resulted in even more efficient queries, scanning less data compared to the partitioned table without clustering.

Partitioning and clustering are two methods of improving performance of data warehouse queries.

Partition for 2020-01-14 (Clustered by VendorID)

| Row | VendorID | Pickup Date | Trip Distance |
|-----|----------|-------------|---------------|
| 3   | 1        | 2020-01-14  | 0.43          |
| 2   | 2        | 2020-01-14  | 0.99          |
| 1   | 3        | 2020-01-14  | 1.2           |

# Practical Impact
To showcase the effectiveness of these methods, I performed queries on non-partitioned, partitioned, and partitioned-clustered tables. The results clearly showed a reduction in the amount of data scanned – moving from 1,6 GB with the non-partitioned table to around 106 MB with the partitioned table, and even less with the partitioned-clustered table.

These optimizations are pivotal in data warehousing, particularly in a cloud environment like BigQuery, where performance improvements can also lead to cost savings.

```sql
-- Check yellow trip data
SELECT * FROM `dtc-de-zoomcamp-410523.ny_taxi.external_yellow_tripdata`;

-- Create a non partitioned table from external table
CREATE OR REPLACE TABLE `dtc-de-zoomcamp-12345.ny_taxi.yellow_tripdata_non_partitoned` 
AS SELECT * FROM `dtc-de-zoomcamp-12345.ny_taxi.external_yellow_tripdata`;

-- Create a partitioned table from external table
CREATE OR REPLACE TABLE `dtc-de-zoomcamp-12345.ny_taxi.yellow_tripdata_partitoned` 
PARTITION BY
  DATE(tpep_pickup_datetime) AS
SELECT * FROM `dtc-de-zoomcamp-12345.ny_taxi.external_yellow_tripdata`;

-- Impact of partition
-- Scanning 1.6GB of data
SELECT DISTINCT(VendorID)
FROM `dtc-de-zoomcamp-12345.ny_taxi.yellow_tripdata_non_partitoned`
WHERE DATE(tpep_pickup_datetime) BETWEEN '2019-06-01' AND '2019-06-30';

-- Scanning ~106 MB of DATA
SELECT DISTINCT(VendorID)
FROM `dtc-de-zoomcamp-12345.ny_taxi.yellow_tripdata_partitoned` 
WHERE DATE(tpep_pickup_datetime) BETWEEN '2019-06-01' AND '2019-06-30';

-- Let's look into the partitons
SELECT table_name, partition_id, total_rows
FROM `ny_taxi.INFORMATION_SCHEMA.PARTITIONS`
WHERE table_name = 'yellow_tripdata_partitoned'
ORDER BY total_rows DESC;

-- Creating a partition and cluster table
CREATE OR REPLACE TABLE `dtc-de-zoomcamp-12345.ny_taxi.yellow_tripdata_partitoned_clustered`
PARTITION BY DATE(tpep_pickup_datetime)
CLUSTER BY VendorID AS
SELECT * FROM `dtc-de-zoomcamp-12345.ny_taxi.external_yellow_tripdata`;

-- Query scans 1.1 GB
SELECT count(*) as trips
FROM `dtc-de-zoomcamp-12345.ny_taxi.yellow_tripdata_partitoned` 
WHERE DATE(tpep_pickup_datetime) BETWEEN '2019-06-01' AND '2020-12-31'
  AND VendorID=1;

-- Query scans 864.5 MB
SELECT count(*) as trips
FROM `dtc-de-zoomcamp-12345.ny_taxi.yellow_tripdata_partitoned_clustered`
WHERE DATE(tpep_pickup_datetime) BETWEEN '2019-06-01' AND '2020-12-31'
  AND VendorID=1;
```
# Machine Learning Model

Understanding data science and machine learning is really important for data engineers. 
Data engineers have a big role in providing good datasets that data scientists need to make their models. 
In Week 3 of this course, I learned about making a simple type of model called linear regression using data about New York taxis. 

```sql
-- SELECT THE COLUMNS OF INTEREST
SELECT passenger_count, trip_distance, PULocationID, DOLocationID, payment_type, fare_amount, tolls_amount, tip_amount
FROM `dtc-de-zoomcamp-12345.ny_taxi.yellow_tripdata_partitoned` WHERE fare_amount != 0;

-- CREATE A ML TABLE WITH APPROPRIATE TYPE
CREATE OR REPLACE TABLE `dtc-de-zoomcamp-12345.ny_taxi.yellow_tripdata_ml` (
`passenger_count` INT64,
`trip_distance` FLOAT64,
`PULocationID` STRING,
`DOLocationID` STRING,
`payment_type` STRING,
`fare_amount` FLOAT64,
`tolls_amount` FLOAT64,
`tip_amount` FLOAT64
) AS (
SELECT CAST(passenger_count AS INT64), trip_distance, cast(PULocationID AS STRING), CAST(DOLocationID AS STRING),
CAST(payment_type AS STRING), fare_amount, tolls_amount, tip_amount
FROM `dtc-de-zoomcamp-12345.ny_taxi.yellow_tripdata_partitoned` WHERE fare_amount != 0
);

-- CREATE MODEL WITH DEFAULT SETTING
CREATE OR REPLACE MODEL `dtc-de-zoomcamp-12345.ny_taxi.tip_model`
OPTIONS
(model_type='linear_reg',
input_label_cols=['tip_amount'],
DATA_SPLIT_METHOD='AUTO_SPLIT') AS
SELECT
*
FROM
`dtc-de-zoomcamp-410523.ny_taxi.yellow_tripdata_ml`
WHERE
tip_amount IS NOT NULL;

-- CHECK FEATURES
SELECT * FROM ML.FEATURE_INFO(MODEL `dtc-de-zoomcamp-12345.ny_taxi.tip_model`);

-- EVALUATE THE MODEL
SELECT
*
FROM
ML.EVALUATE(MODEL `dtc-de-zoomcamp-12345.ny_taxi.tip_model`,
(
SELECT
*
FROM
`dtc-de-zoomcamp-12345.ny_taxi.yellow_tripdata_ml`
WHERE
tip_amount IS NOT NULL
));

-- PREDICT THE MODEL
SELECT
*
FROM
ML.PREDICT(MODEL `dtc-de-zoomcamp-12345.ny_taxi.tip_model`,
(
SELECT
*
FROM
`dtc-de-zoomcamp-12345.ny_taxi.yellow_tripdata_ml`
WHERE
tip_amount IS NOT NULL
));

-- PREDICT AND EXPLAIN
SELECT
*
FROM
ML.EXPLAIN_PREDICT(MODEL `dtc-de-zoomcamp-12345.ny_taxi.tip_model`,
(
SELECT
*
FROM
`dtc-de-zoomcamp-410523.ny_taxi.yellow_tripdata_ml`
WHERE
tip_amount IS NOT NULL
), STRUCT(3 as top_k_features));

-- HYPER PARAM TUNNING
CREATE OR REPLACE MODEL `dtc-de-zoomcamp-12345.ny_taxi.tip_hyperparam_model`
OPTIONS
(model_type='linear_reg',
input_label_cols=['tip_amount'],
DATA_SPLIT_METHOD='AUTO_SPLIT',
num_trials=5,
max_parallel_trials=2,
l1_reg=hparam_range(0, 20),
l2_reg=hparam_candidates([0, 0.1, 1, 10])) AS
SELECT
*
FROM
`dtc-de-zoomcamp-12345.ny_taxi.yellow_tripdata_ml`
WHERE
tip_amount IS NOT NULL;
```
