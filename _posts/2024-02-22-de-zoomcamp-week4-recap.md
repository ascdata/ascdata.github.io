---
title:  "Data Engineering Zoomcamp Week 4: Analytics Engineering and Data Visualization"
mathjax: true
layout: post
categories: media
---

# What is Analytics Engineering?

As the data domain has developed over time, new tools have been introduced that have changed the dynamics of working with data:

1. Massively parallel processing (MPP) databases
    * Lower the cost of storage 
    * BigQuery, Snowflake, Redshift, etc.
1. Data-pipelines-as-a-service
    * Simplify the ETL process
    * Fivetran, Stitch, etc.
1. SQL-first / Version control systems
    * Looker, etc.
1. Self service analytics
    * Mode, etc.
1. Data governance

The introduction of all of these tools changed the way the data teams work as well as the way that the stakeholders consume the data, creating a gap in the roles of the data team. Traditionally:

* The ***data engineer*** prepares and maintains the infrastructure the data team needs.
* The ***data analyst*** uses data to answer questions and solve problems.
* The ***data scientist*** predicts the future based on past patterns and covers the what-ifs.

However, with the introduction of these tools, both data scientists and analysts find themselves writing more code even though they're not software engineers and writing code isn't their top priority.  Data engineers are good software engineers but they don't have the training in how the data is going to be used  by the business users.

The ***analytics engineer*** is the role that tries to fill the gap: it introduces the good software engineering practices to the efforts of data analysts and data scientists. The analytics engineer may be exposed to the following tools:
1. Data Loading (Stitch...)
1. Data Storing (Data Warehouses)
1. Data Modeling (dbt, Dataform...)
1. Data Presentation (BI tools like Looker, Mode, Tableau...)

This lesson focuses on the last two parts: Data Modeling and Data Presentation.

# Data Modeling Concepts

## ETL vs. ELT

When ingesting data, Data Warehouses use the Export, Transform and Load (ETL) model whereas Data Lakes use Export, Load and Transform (ELT).

The main difference between them is the order of steps. In Data Warehouses, ETL means the data is transformed (preprocessed, etc.) before arriving to its final destination, whereas in Data Lakes, ELT the data is directly stored without any transformations and any schemas are derived when reading the data from the Data Lake.

![etl-vs-elt](https://raw.githubusercontent.com/ascdata/ascdata.github.io/master/_posts/media/etl-vs-elt.png)

## Dimensional Modeling

Ralph Kimball's Dimensional Modeling is an approach to Data Warehouse design which focuses on two main points:
* Deliver data which is understandable to the business users
* Deliver fast query performance

Other goals such as reducing redundant data (prioritized by other approaches like Bill Inmon) are secondary to these goals. Dimensional Modeling also differs from other approaches to Data Warehouse design such as Data Vaults.

Dimensional Modeling is based around two important concepts:
* ***Fact Table***:
    * _Facts_ = _Measures_
    * Typically numeric values which can be aggregated, such as measurements or metrics
        * Examples: sales, orders, etc.
    * Corresponds to a business process
    * Can be thought of as "verbs"
* ***Dimension Table***:
    * _Dimension_ = _Context_
    * Groups of hierarchies and descriptors that define the facts
        * Example: customer, product, etc.
    * Corresponds to a business entity.
    * Can be thought of as "nouns"
* Dimensional Modeling is built on a star schema with fact tables surrounded by dimension tables

A good way to understand the architecture of Dimensional Modeling is by drawing an analogy between dimensional modeling and a restaurant:

* Stage Area:
    * Contains the raw data
    * Not meant to be exposed to everyone
    * Similar to the food storage area in a restaurant
* Processing area:
    * From raw data to data models
    * Focuses in efficiency and ensuring standards
    * Similar to the kitchen in a restaurant
* Presentation area:
    * Final presentation of the data
    * Exposure to business stakeholder
    * Similar to the dining room in a restaurant

# What is "dbt"? 
DBT (Data Build Tool) is an open-source software application that enables data analysts and engineers to transform data in their warehouses more effectively. 
It acts as a workflow tool that allows teams to easily define, run and test data modelling code, primarily written in SQL, to organise, cleanse and transform their data.

Using DBT over writing SQL manually for data transformations in a data warehouse has several advantages that improve efficiency, scalability and reliability of data workflows. 
While SQL is powerful for querying and manipulating data, DBT adds a layer of functionality that addresses common challenges in data engineering and analytics projects.

# How to use dbt?

dbt has two main components: _dbt Core_ and _dbt Cloud_:
* ***dbt Core***: open-source project that allows the data transformation
    * Builds and runs a dbt project (.sql and .yaml files)
    * Includes SQL compilation logic, macros and database adapters
    * Includes a CLI interface to run dbt commands locally
* ***dbt Cloud***: SaaS application to develop and manage dbt projects
    * Web-based IDE to develop, run and test a dbt project
    * Jobs orchestration
    * Logging and alerting
    * Intregrated documentation

I will use the dbt Cloud IDE.

![dbt](https://raw.githubusercontent.com/ascdata/ascdata.github.io/master/_posts/media/dbt.png)

# Anatomy of a dbt model

dbt models are mostly written in SQL but they also make use of the [Jinja templating language](https://jinja.palletsprojects.com/en/3.0.x/) for templates. 

Here's an example dbt model:

```sql
{{
    config(materialized='table')
}}

SELECT *
FROM staging.source_table
WHERE record_state = 'ACTIVE'
```

* In the Jinja statement defined within the `{{ }}` block I call the [`config()` function](https://docs.getdbt.com/reference/dbt-jinja-functions/config).
* I commonly use the `config()` function at the beginning of a model to define a ***materialization strategy***: a strategy for persisting dbt models in a warehouse.
    * The `table` strategy means that the model will be rebuilt as a table on each run.
    * The `view` strategy would rebuild the model on each run as a SQL view.
    * The `incremental` strategy is essentially a `table` strategy but it allows us to add or update records incrementally rather than rebuilding the complete table on each run.
    * The `ephemeral` strategy creates a Common Table Expression.
  
dbt will compile this code into the following SQL query:

```sql
CREATE TABLE my_schema.my_model AS (
    SELECT *
    FROM staging.source_table
    WHERE record_state = 'ACTIVE'
)
```

After the code is compiled, dbt will run the compiled code in the Data Warehouse. Additional model properties are stored in YAML files. Traditionally, these files were named `schema.yml`.

# The FROM clause

The `FROM` clause within a `SELECT` statement defines the _sources_ of the data to be used.

The following sources are available to dbt models:

* ***Sources***: The data loaded within the Data Warehouse.
    * This data can be accessed with the `source()` function.
    * The `sources` key in othe YAML file contains the details of the databases that the `source()` function can access and translate into proper SQL-valid names.
        * Additionally, "source freshness" can be defined to each source to check whether a source is "fresh" or "stale", which can be useful to check whether the data pipelines are working properly.

* ***Seeds***: CSV files which can be stored in the repo under the `seeds` folder.
    * The repo gives version controlling along with all of its benefits.
    * Seeds are best suited to static data which changes infrequently.
    * Seed usage:
        1. Add a CSV file to the `seeds` folder.
        2. Run the `dbt seed` to create a table in the Data Warehouse.
        3. Refer to the seed in the model with the `ref()` function.

Here's an example of a source in a `.yml` file:

```yaml
sources:
    - name: staging
      database: production
      schema: trips_data_all

      loaded_at_field: record_loaded_at
      tables:
        - name: green_tripdata
        - name: yellow_tripdata
          freshness:
            error_after: {count: 6, period: hour}
```

And here's how to reference a source in a `FROM` clause:

```sql
FROM {{ source('staging','yellow_tripdata') }}
```
* The first argument of the `source()` function is the source name, and the second is the table name.

In this case the `taxi_zone_lookup.csv` file in my `seeds` folder contains `locationid`, `borough`, `zone` and `service_zone`:

```sql
SELECT
    locationid,
    borough,
    zone,
    replace(service_zone, 'Boro', 'Green') as service_zone
FROM {{ ref('taxi_zone_lookup) }}
```

The `ref()` function references underlying tables and views in the Data Warehouse. When compiled, it will automatically build the dependencies and resolve the correct schema. 
So, if BigQuery contains a schema/dataset called `dbt_dev` inside the database which I'm using for development and it contains a table called `stg_green_tripdata`, then the following code...

```sql
WITH green_data AS (
    SELECT *,
        'Green' AS service_type
    FROM {{ ref('stg_green_tripdata') }}
),
```

...will compile to this:

```sql
WITH green_data AS (
    SELECT *,
        'Green' AS service_type
    FROM "my_project"."dbt_dev"."stg_green_tripdata"
),
```
* The `ref()` function translates the references table into the full reference, using the `database.schema.table` structure.

# Defining a source and creating a model

I created two new folders under my `models` folder:
* `staging` will have the raw models.
* `core` will have the models that we will expose at the end to the BI tool.

Under `staging` I will add two new files: `sgt_green_tripdata.sql` and `schema.yml`:
```yaml
# schema.yml

version: 2

sources:
    - name: staging
      database: your_project
      schema: trips_data_all

      tables:
          - name: green_tripdata
          - name: yellow_tripdata
```
* I defined the ***sources*** in the `schema.yml` model properties file.
* I defined the two tables for yellow and green taxi data as my sources.
```sql
-- sgt_green_tripdata.sql

{{ config(materialized='view') }}

select * from {{ source('staging', 'green_tripdata') }}
limit 100
```
* This query will create a ***view*** in the `staging` dataset/schema in the database.
* I made use of the `source()` function to access the green taxi data table, which is defined inside the `schema.yml` file.

The advantage of having the properties in a separate file is that I can easily modify the `schema.yml` file to change the database details and write to different databases without having to modify my `sgt_green_tripdata.sql` file.

The model can be run with the `dbt run` command.

# Macros

***Macros*** are pieces of code in Jinja that can be reused, similar to functions in other languages.

dbt already includes a series of macros like `config()`, `source()` and `ref()`, but custom macros can also be defined.

Macros allows to add features to SQL that are not otherwise available, such as:
* Use control structures such as `if` statements or `for` loops
* Use environment variables
* Abstract snippets of SQL into reusable macros

Macros are defined in separate .sql files.

Here is a macro definition example:

```sql
{# This macro returns the description of the payment_type #}

{% macro get_payment_type_description(payment_type) %}

    case {{ payment_type }}
        when 1 then 'Credit card'
        when 2 then 'Cash'
        when 3 then 'No charge'
        when 4 then 'Dispute'
        when 5 then 'Unknown'
        when 6 then 'Voided trip'
    end

{% endmacro %}
```
* The macro keyword states that the line is a macro definition. It includes the name of the macro as well as the parameters.
* The code of the macro itself goes between two statement delimiters. The second statement delimiter contains an endmacro keyword.

Here's how to use the macro:
```sql
select
    {{ get_payment_type_description('payment-type') }} as payment_type_description,
    congestion_surcharge::double precision
from {{ source('staging','green_tripdata') }}
where vendorid is not null
```
* It passes a `payment-type` variable which may be an integer from 1 to 6.

And this is what it would compile to:
```sql
select
    case payment_type
        when 1 then 'Credit card'
        when 2 then 'Cash'
        when 3 then 'No charge'
        when 4 then 'Dispute'
        when 5 then 'Unknown'
        when 6 then 'Voided trip'
    end as payment_type_description,
    congestion_surcharge::double precision
from {{ source('staging','green_tripdata') }}
where vendorid is not null
```
* The macro is replaced by the code contained within the macro definition.
