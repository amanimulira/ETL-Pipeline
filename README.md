This is a breakdown of the complete process of constructing a modern ELT (Extract-Load-Transform) pipeline using:

- Dbt (Data Build Tool) for transformations,
- Snowflake as the cloud data warehouse, and 
- Apache Airflow for workflow orchestration. 

Unlike traditional ETL, where data is transformed before it’s loaded, ELT loads raw data into the warehouse first and then transforms it in place. This pattern is ideal for cloud warehousing because it takes advantage of scalable compute and makes transformation independent of data ingestion tooling.

## The Data Stack:

<img width=auto height="100" alt="image" src="https://github.com/user-attachments/assets/e0facbf1-d4e7-496c-be94-9d0d729fb692" />

<img width=auto height="100" alt="image" src="https://github.com/user-attachments/assets/988c0eb6-18db-4a09-b7a7-82de277442bf" />

<img width=auto height="100" alt="image" src="https://github.com/user-attachments/assets/a43c2449-32aa-485c-9c1f-bef9ae739184" />

dbt lets us transform data in SQL with software engineering principles such as modularity, tests, version control, and documentation built in. It turns SQL queries into reusable, tested data models.

Snowflake is a cloud-native data warehouse that separates compute from storage, allowing dynamic scaling and cost-efficient processing of large datasets – great for running dbt models.
Apache airflow orchestrates tasks in a DAG (Directed Acyclic Graph), ensuring that steps like ingest → transform → test → deploy run reliably, monitorably, and on a schedule.

Together, they form a modern data stack which can be used in analytics engineering and data platforms. 


## Pipeline Architecture (Step-by-Step)

### Step 1 - data Ingestion (Extract & Load)

First Define data sources could be CSVs, APIs, logs, or external databases. Then Load raw data into Snowflake. Airflow triggers tasks (custom code or operator) that insert raw datasets into Snowflake staging schemas. Loading raw data first creates a historical source of truth that can be traced and debugged later — an important part of data governance.

### Step 2 - dbt setup & project initialization 

Install and configure dbt.

- dbt init project_name
- Set up a profiles.yml with Snowflake credentials and warehouse settings.

Build folder structure:

- models/staging/ for raw table cleanups
- models/marts/ for business-ready analytics tables
- macros/ for reusable business logic.

Define sources (sources.yml).

- This tells dbt which Snowflake tables are your raw inputs.

Step 3 - Writing Transformation

dbt models are just SQL files with templating to orchestrate transformations.

```

-- models/staging/stg_orders.sql
{{ config(materialized='view') }}

SELECT
  order_id,
  customer_id,
  order_date,
  amount
FROM {{ source('raw_schema','orders') }}
WHERE order_date >= '2025-01-01';

```

<img width="807" height="243" alt="Found 3 data tests, 5 models, 2 sources, 637 nacros" src="https://github.com/user-attachments/assets/feef784e-d9ba-40a8-987d-923927074406" />

Step 4 - Testing & Documentation

dbt allows for tests to be defined in .yml files to enforce uniqueness, not-null constraints, and referential integrity.

These tests are critical — especially in production — because they catch bad assumptions earlier.


<img width="400" height="93" alt="cosmos" src="https://github.com/user-attachments/assets/e040c9fb-bf0e-47fb-ad40-cbe030d2239e" />

<img width="250" height="250" alt="docker" src="https://github.com/user-attachments/assets/035adc5b-a646-4223-a819-0e0f270dc717" />

Cosmos converts dbt projects into native Airflow DAG.
Instead of treating dbt as a single opaque command (dbt run), Cosmos:
Parses your dbt project
Reads ref() dependencies
Translates dbt models into individual Airflow tasks
Preserves dbt’s DAG inside Airflow
This gives you true orchestration, not just scheduling.
Then docker is used as cosmos alone isn’t enough. dbt still needs, A specific Python version, Specific dbt + adapter versions, Snowflake connector compatibility, and consistent filesystem paths
Docker solves this by locking the runtime environment.
Airflow itself runs inside Docker, and dbt runs in a dedicated dbt container mounted into Airflow.

<img width="1600" height="657" alt="airflow" src="https://github.com/user-attachments/assets/abff65c4-581e-4a6c-8818-d87923e6a14a" />




