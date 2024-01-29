---
title:  "Data Warehouse vs. Data Lake"
mathjax: true
layout: post
categories: media
---

 ![Data Warehouse vs. Data Lake](https://raw.githubusercontent.com/ascdata/ascdata.github.io/master/_posts/media/datawarehouse-vs-datalake.JPG)

A Data Lake should not be mistaken for a Data Warehouse, as there are notable distinctions:

__Data Processing__:  
Data Lake: The data remains raw and undergoes minimal processing, typically being unstructured.  
Data Warehouse: Data in the is refined through cleaning, pre-processing, and structuring for specific use cases

__Data Size__:  
Data Lake: Data Lakes are extensive, containing vast amounts of data. Data is transformed when in use only and can be stored indefinitely. 
Data Warehouse: Data Warehouses are small in comparison with Data Lakes. Data is always preprocessed before ingestion and may be purged periodically.  

__Structure__:  
Data Lake: Data is undefined and adaptable for a wide variety of purposes.  
Data Warehouse: Data is historical and relational, often associated with transaction systems.  

__Users__:  
Data Lake: Primarily utilized by data scientists and data analysts.  
Data Warehouse: Mainly employed by business analysts. 

__Use cases__:  
Data Lake: Stream processing, machine learning, real-time analytics, among others.  
Data Warehouse: Ideal for batch processing, business intelligence, and reporting.  

Data Lakes came into existence because as companies started to realize the importance of data, they soon found out that they couldn't ingest data right away into their Data Warehouses but they didn't want to waste uncollected data when their devs hadn't yet finished developing the necessary relationships for a Data Warehouse, so the Data Lake was born to collect any potentially useful data that could later be used in later steps from the very start of any new projects.

When ingesting data, Data Warehouses use the Export, Transform and Load (ETL) model whereas Data Lakes use Export, Load and Transform (ELT).

The main difference between them is the order of steps. In Data Warehouses, ETL means the data is transformed before arriving at its final destination, whereas in Data Lakes, the data is directly stored without any transformations and is transformed afterward.
