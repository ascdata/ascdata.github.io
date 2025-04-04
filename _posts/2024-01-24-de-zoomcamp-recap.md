---
title:  "Data Engineering Zoomcamp: GCP, Docker, PostgreSQL, Jupyter Notebook, Terraform"
mathjax: true
layout: post
categories: media
---

The Data Engineering Zoomcamp is a practical, self-paced course offered by [DataTalks.club](https://datatalks.club/), where I will set up and develop a cloud data infrastructure from scratch. My focus will be on developing data engineering and problem-solving skills with mutual support from the DataTalks community. The course spans 6 weeks and concludes with a practical project, which will be assessed to determine the issuance of my certificate. The contents of the first week include:

- Cloud Infrastructure (GCP)
- Containerization (Docker)
- Databases (PostgreSQL, SQL)
- Development (Python, Jupyter Notebook)
- Infrastructure as Code (Terraform)

The complete course will also treat the following tools:
- [Docker](https://www.docker.com/) is a platform to automate the deployment of applications within lightweight, portable containers
- [PostgreSQL](https://www.postgresql.org/) is an open-source relational database management system
- [Mage](https://www.mage.ai/) is a data pipeline tool for transforming and integrating data
- [Google BigQuery](https://cloud.google.com/bigquery/) is serverless data warehouse
- [dbt](https://discover.getdbt.com/free-account/) is a tool that focuses on transforming and modeling data within a data warehouse
- [Apache Spark](https://spark.apache.org/) is a analytics engine for large-scale data processing
- [Apache Kafka](https://kafka.apache.org/) is an event streaming platform designed for high-throughput, fault-tolerant, and scalable data streaming

  ![Architecture](https://raw.githubusercontent.com/ascdata/ascdata.github.io/master/_posts/media/course-architecture.jpg)

## Initial setup with GCP and Docker
After setting up a GCP instance with Debian as the operating system, I initially installed the Python distribution Anaconda, which incidentally includes Jupyter Notebook. In the next step, I installed Docker and Docker-Compose, and concurrently deployed the Postgres database system and pgadmin in two containers (with docker-compose). It is important to note at this point that both containers are in the same Docker network.

```
docker network create pg-network
```

```
services:
  pgdatabase:
    image: postgres:13
    environment:
      - POSTGRES_USER=root
      - POSTGRES_PASSWORD=root
      - POSTGRES_DB=ny_taxi
    volumes:
      - "$(pwd)/ny_taxi_postgres_data:/var/lib/postgresql/data"
    ports:
      - "5432:5432"
  pgadmin:
    image: dpage/pgadmin4
    environment:
      - PGADMIN_DEFAULT_EMAIL=admin@admin.com
      - PGADMIN_DEFAULT_PASSWORD=root
    ports:
      - "8080:80"
 ```
Docker containers are stateless. Any changes done inside a container will **not** be saved when the container is killed and started again. As you can see I use volume mapping for the PostgreSQL database files to prevent data loss. Additionally, there are some necessary environment variables, such as user, password, database name, port, etc.


## Creating a custom pipeline
This is a dummy pipeline.py script that receives an argument and prints it.
```python
import sys
import pandas

# print arguments
print(sys.argv)

# argument 0 is the name os the file
# argumment 1 contains the actual first argument we care about
day = sys.argv[1]

# print a sentence with the argument
print(f'job finished successfully for day = {day}')
```
With the following dockerfile, I containerized this script.

```
FROM python:3.9.1

# installing prerequisites
RUN pip install pandas

# set up the working directory inside the container
WORKDIR /app
# copy the script to the container. 1st name is source file, 2nd is destination
COPY pipeline.py pipeline.py

# define what to do first when the container runs (run the script)
ENTRYPOINT ["python", "pipeline.py"]
```

With the pipeline.py script and the dockerfile in the same directory I build the image with ```docker build``` and run it with an argument, so the pipeline will receice it (```docker run -it pipeline 2```).

## Scripting

This script utilizes the argparse module to handle command-line arguments and employs the wget command to download a CSV file containing information about taxi trips in New York. Then it utilizes the SQLAlchemy library to establish a connection to the PostgreSQL database and incrementally reads the CSV file into Pandas DataFrames with a chunk size of 100.000 rows and writes each chunk into the PostgreSQL database. Additionally, the date values in the 'tpep_pickup_datetime' and 'tpep_dropoff_datetime' columns are converted into Pandas datetime objects.

```python
#!/usr/bin/env python
# coding: utf-8

import os
import argparse

from time import time

import pandas as pd
from sqlalchemy import create_engine


def main(params):
    user = params.user
    password = params.password
    host = params.host 
    port = params.port 
    db = params.db
    table_name = params.table_name
    url = params.url
    
    if url.endswith('.csv.gz'):
        csv_name = 'output.csv.gz'
    else:
        csv_name = 'output.csv'

    os.system(f"wget {url} -O {csv_name}")

    engine = create_engine(f'postgresql://{user}:{password}@{host}:{port}/{db}')

    df_iter = pd.read_csv(csv_name, iterator=True, chunksize=100000)

    df = next(df_iter)

    df.tpep_pickup_datetime = pd.to_datetime(df.tpep_pickup_datetime)
    df.tpep_dropoff_datetime = pd.to_datetime(df.tpep_dropoff_datetime)

    df.head(n=0).to_sql(name=table_name, con=engine, if_exists='replace')

    df.to_sql(name=table_name, con=engine, if_exists='append')


    while True: 

        try:
            t_start = time()
            
            df = next(df_iter)

            df.tpep_pickup_datetime = pd.to_datetime(df.tpep_pickup_datetime)
            df.tpep_dropoff_datetime = pd.to_datetime(df.tpep_dropoff_datetime)

            df.to_sql(name=table_name, con=engine, if_exists='append')

            t_end = time()

            print('inserted another chunk, took %.3f second' % (t_end - t_start))

        except StopIteration:
            print("Finished ingesting data into the postgres database")
            break

if __name__ == '__main__':
    parser = argparse.ArgumentParser(description='Ingest CSV data to Postgres')

    parser.add_argument('--user', required=True, help='user name for postgres')
    parser.add_argument('--password', required=True, help='password for postgres')
    parser.add_argument('--host', required=True, help='host for postgres')
    parser.add_argument('--port', required=True, help='port for postgres')
    parser.add_argument('--db', required=True, help='database name for postgres')
    parser.add_argument('--table_name', required=True, help='name of the table where we will write the results to')
    parser.add_argument('--url', required=True, help='url of the csv file')

    args = parser.parse_args()

    main(args)
```

In the next step I dockerized the script.

```
FROM python:3.9.1

RUN apt-get install wget
# psycopg2 is a postgres db adapter for python: sqlalchemy needs it
RUN pip install pandas sqlalchemy psycopg2

WORKDIR /app
COPY ingest_data.py ingest_data.py 

ENTRYPOINT [ "python", "ingest_data.py" ]
```

```
URL="https://github.com/DataTalksClub/nyc-tlc-data/releases/download/yellow/yellow_tripdata_2021-01.csv.gz"

docker run -it \
  --network=pg-network \
  taxi_ingest:v001 \
    --user=root \
    --password=root \
    --host=pg-database \
    --port=5432 \
    --db=ny_taxi \
    --table_name=yellow_taxi_trips \
    --url=${URL}
```

## Terraform

At first I set up a Terraform configuration. It's important that a Terraform configuration be in its own working directory. The main.tf file is written in the Terraform language and contains all the necessary information to describe basic infrastructure. In this case I hard-coded the credentials path but I don't recommend it.

In the following code there are resource types like google_storage_bucket and google_bigquery_dataset.

- The google_storage_bucket resource type is used to create and configure a Google Cloud Storage Bucket. Google Cloud Storage is an object storage service that allows you to store data as objects in containers called buckets.
- The google_bigquery_dataset resource type is used to create and configure a BigQuery Dataset. Google BigQuery is a fully managed, serverless database service for analyzing large datasets using SQL (Data Warehouse).

```tf
terraform {
  required_providers {
    google = {
      source  = "hashicorp/google"
      version = "5.6.0"
    }
  }
}

provider "google" {
  credentials = "<path/to/authkeys>.json"
  project = "taxi-rides-ny-410014"
  region  = "europe-west1"
}

resource "google_storage_bucket" "tera-bucket" {
  name          = "taxi-rides-ny-410014-terra-bucket"
  location      = "europe-west1"
  force_destroy = true

  lifecycle_rule {
    condition {
      age = 1
    }
    action {
      type = "AbortIncompleteMultipartUpload"
    }
  }
}

resource "google_bigquery_dataset" "terra-dataset" {
  dataset_id = "terra_dataset"

}
```

To submit the Google credentials I assigned an environment variable in my bash shell:

```export GOOGLE_CREDENTIALS="<path/to/authkeys>.json"```

To check the variable, I used:

```echo $GOOGLE_CREDENTIALS```

Then I initialized my work directory by downloading the necessary providers/plugins with the following command within the same directory as main.tf:

```tf
terraform init
```

To create a preview of the changes to be applied against a remote state, I used:

```tf
terraform plan
```

Finally I applied the changes to the infrastructure:

```tf
terraform apply
```

Alternatively it's also possible to outsource the Terraform variables. Therefore I created a variables.tf file with the following content:

```tf
variable "credentials" {
  description = "My Credentials"
  default     = "<path/to/authkeys>.json"
  #ex: if you have a directory where this file is called keys with your service account json file
  #saved there as my-creds.json you could use default = "./keys/my-creds.json"
}


variable "project" {
  description = "Project"
  default     = "taxi-rides-ny-410014"
}

variable "region" {
  description = "Region"
  #Update the below to your desired region
  default     = "europe-west1"
}

variable "location" {
  description = "Project Location"
  #Update the below to your desired location
  default     = "europe-west1"
}

variable "bq_dataset_name" {
  description = "My BigQuery Dataset Name"
  #Update the below to what you want your dataset to be called
  default     = "terra_dataset"
}

variable "gcs_bucket_name" {
  description = "My Storage Bucket Name"
  #Update the below to a unique bucket name
  default     = "taxi-rides-ny-410014-terra-bucket"
}

variable "gcs_storage_class" {
  description = "Bucket Storage Class"
  default     = "STANDARD"
}
```

I also had to modify the main.tf so that the variables will be used:

```tf
terraform {
  required_providers {
    google = {
      source  = "hashicorp/google"
      version = "5.6.0"
    }
  }
}

provider "google" {
  credentials = file(var.credentials)
  project     = var.project
  region      = var.region
}


resource "google_storage_bucket" "terra-bucket" {
  name          = var.gcs_bucket_name
  location      = var.location
  force_destroy = true


  lifecycle_rule {
    condition {
      age = 1
    }
    action {
      type = "AbortIncompleteMultipartUpload"
    }
  }
}

resource "google_bigquery_dataset" "terra_dataset" {
  dataset_id = var.bq_dataset_name
  location   = var.location
}
```

That was the content of my first week of the Data Engineering Zoomcamp. I have already learned a lot and look forward to the upcoming weeks!
