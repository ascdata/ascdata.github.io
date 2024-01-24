---
title:  "Data Engineering Zoomcamp: Week 1 Recap"
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

  ![Architecture](https://raw.githubusercontent.com/ascdata/ascdata.github.io/master/_posts/media/course-architecture.jpg)

## Initial setup with GCP and Docker
After setting up a GCP instance with Debian as the operating system, I initially installed the Python distribution Anaconda, which incidentally includes Jupyter Notebook. In the next step, I installed Docker and Docker-Compose, and concurrently deployed the Postgres database system and pgadmin in two containers. It is important to note at this point that both containers are in the same Docker network.

```
docker network create pg-network
```

```
docker run -it \
  -e POSTGRES_USER="root" \
  -e POSTGRES_PASSWORD="root" \
  -e POSTGRES_DB="ny_taxi" \
  -v $(pwd)/ny_taxi_postgres_data:/var/lib/postgresql/data \
  -p 5432:5432 \
  --network=pg-network \
  --name pg-database \
  postgres:13
```
As you can see I use volume mapping for the PostgreSQL database files to prevent data loss.

```
docker run -it \
  -e PGADMIN_DEFAULT_EMAIL="admin@admin.com" \
  -e PGADMIN_DEFAULT_PASSWORD="root" \
  -p 8080:80 \
  --network=pg-network \
  --name pgadmin-2 \
  dpage/pgadmin4
 ```
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
    
    # the backup files are gzipped, and it's important to keep the correct extension
    # for pandas to be able to open the file
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
docker build -t taxi_ingest:v001 .
```

## Terraform

To be continued
