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

To be continued.
