# 🏦 Banking Modern Data Stack

![Snowflake](https://img.shields.io/badge/Snowflake-29B5E8?logo=snowflake&logoColor=white)
![dbt](https://img.shields.io/badge/dbt-FF694B?logo=dbt&logoColor=white)
![Apache Airflow](https://img.shields.io/badge/Apache%20Airflow-017CEE?logo=apacheairflow&logoColor=white)
![Apache Kafka](https://img.shields.io/badge/Apache%20Kafka-231F20?logo=apachekafka&logoColor=white)
![Debezium](https://img.shields.io/badge/Debezium-EF3B2D?logo=apache&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-336791?logo=postgresql&logoColor=white)
![MinIO](https://img.shields.io/badge/MinIO-C72E29?logo=minio&logoColor=white)
![Python](https://img.shields.io/badge/Python-3776AB?logo=python&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?logo=docker&logoColor=white)
![Power BI](https://img.shields.io/badge/Power%20BI-F2C811?logo=powerbi&logoColor=black)

---

## 📌 Project Overview

This project demonstrates an **end-to-end modern data stack for a Banking domain**.

The project simulates banking data using Python Faker, stores the transactional data in PostgreSQL, captures database changes using Debezium and Kafka, stores CDC events in MinIO, loads the data into Snowflake using Apache Airflow, and transforms the data using dbt into analytics-ready models.

The project also demonstrates **SCD Type-2 historical tracking** using dbt snapshots and visualizes the final analytics data using Power BI.

---

## 🏗️ Architecture

<img width="1746" height="901" alt="Architecture" src="https://github.com/user-attachments/assets/0f524197-2f03-4c5e-9bde-9318100e2035" />


### Pipeline Flow

```text
Python Faker Data Generator
            │
            ▼
       PostgreSQL
          (OLTP)
            │
            ▼
        Debezium
           CDC
            │
            ▼
          Kafka
            │
            ▼
    Python Kafka Consumer
            │
            ▼
          MinIO
      Raw Parquet Data
            │
            ▼
         Airflow
      Orchestration
            │
            ▼
       Snowflake RAW
            │
            ▼
           dbt
            │
     ┌──────┼──────┐
     ▼      ▼      ▼
 Staging  Marts   SCD2
     │      │      │
     └──────┼──────┘
            ▼
   Snowflake ANALYTICS
            │
            ▼
         Power BI
```

---

## ⚙️ Tech Stack

| Technology | Purpose |
|---|---|
| PostgreSQL | Source OLTP database |
| Python + Faker | Synthetic banking data generation |
| Debezium | Change Data Capture |
| Apache Kafka | Event streaming |
| Python Consumer | Kafka CDC event consumption |
| MinIO | S3-compatible raw object storage |
| Apache Airflow | Pipeline orchestration |
| Snowflake | Cloud data warehouse |
| dbt | Data transformation and SCD2 snapshots |
| Power BI | Analytics and visualization |
| Docker | Containerized infrastructure |

---

# 📊 Data Model

The banking system contains three main source tables.

### Customers

```text
customers
├── id
├── first_name
├── last_name
├── email
└── created_at
```

### Accounts

```text
accounts
├── account_id
├── customer_id
├── account_type
├── balance
└── created_at
```

### Transactions

```text
transactions
├── transaction_id
├── account_id
├── amount
├── txn_type
├── related_account_id
├── status
└── created_at
```

---

# 1️⃣ Data Generation

Synthetic banking data is generated using **Python Faker**.

The generator creates:

- Customers
- Accounts
- Transactions

The final demonstration dataset contains:

```text
500 Customers
1000 Accounts
2500 Transactions
```

Data is inserted into PostgreSQL, which acts as the source OLTP system.

### Data Generator

File:

```text
data-generator/faker_generator.py
```

### Flow

```text
Python Faker
     │
     ▼
PostgreSQL
     │
     ├── Customers
     ├── Accounts
     └── Transactions
```

### 📸 Data Generator Proof

<img width="1917" height="1026" alt="producer" src="https://github.com/user-attachments/assets/973f8f88-4dd4-4aad-85cc-30b6c9548337" />


---

# 2️⃣ PostgreSQL – Source OLTP

PostgreSQL acts as the transactional source database.

The generated banking data is stored in PostgreSQL before CDC processing.

Main tables:

```text
customers
accounts
transactions
```

PostgreSQL provides the source data that is captured by Debezium.

---

# 3️⃣ Debezium + Kafka – Change Data Capture

Debezium is used to capture changes from PostgreSQL.

The PostgreSQL changes are published to Kafka topics.

### Kafka Topics

```text
banking_server.public.customers
banking_server.public.accounts
banking_server.public.transactions
```

### CDC Flow

```text
PostgreSQL
     │
     ▼
Debezium
     │
     ▼
Kafka Topics
```

This allows database changes to be streamed from PostgreSQL to downstream systems.

### 📸 Debezium / Kafka Proof

<img width="1917" height="1026" alt="kafka" src="https://github.com/user-attachments/assets/345c0434-c43d-4a3a-9942-47d5d7f85a5c" />


---

# 4️⃣ Kafka → MinIO Consumer

A Python Kafka consumer reads CDC events from Kafka.

File:

```text
consumer/kafka_to_minio.py
```

The consumer performs the following steps:

1. Connects to Kafka
2. Reads CDC events
3. Extracts the `after` record
4. Converts records into Parquet
5. Uploads Parquet files to MinIO

### Flow

```text
Kafka
  │
  ▼
Python Consumer
  │
  ▼
Parquet
  │
  ▼
MinIO
```

### 📸 Consumer Proof

<img width="1917" height="1021" alt="consumer" src="https://github.com/user-attachments/assets/a8d20f2a-5a2c-43f0-9eec-4f27591720c2" />


---

# 5️⃣ MinIO – Raw Storage

MinIO provides S3-compatible object storage for raw CDC data.

The Kafka consumer stores the CDC records as Parquet files.

Example structure:

```text
raw/
├── customers/
│   └── date=YYYY-MM-DD/
│       └── *.parquet
│
├── accounts/
│   └── date=YYYY-MM-DD/
│       └── *.parquet
│
└── transactions/
    └── date=YYYY-MM-DD/
        └── *.parquet
```

### 📸 MinIO Proof

<img width="1917" height="1025" alt="minio" src="https://github.com/user-attachments/assets/4d974866-7d18-490f-a922-44509d8b21a6" />


---

# 6️⃣ Apache Airflow – Orchestration

Apache Airflow is used to orchestrate the pipeline.

The main DAGs are:

```text
minio_to_snowflake_banking
SCD2_snapshots
```

---

## MinIO → Snowflake DAG

The DAG loads Parquet data from MinIO into Snowflake RAW.

```text
MinIO
   │
   ▼
Download Parquet
   │
   ▼
Snowflake Stage
   │
   ▼
COPY INTO
   │
   ▼
Snowflake RAW
```

---

## SCD2 DAG

The SCD2 DAG runs dbt models followed by dbt snapshots.

```text
dbt_run
   │
   ▼
dbt_snapshot
```

This ensures that the dbt models are executed before the historical snapshots are updated.

### 📸 Airflow Proof

<img width="1917" height="1028" alt="Airflow" src="https://github.com/user-attachments/assets/b19ad237-1cdf-4a66-b208-87faaace688f" />
<img width="1917" height="1023" alt="Airflow1" src="https://github.com/user-attachments/assets/5bbba805-a976-4b4b-9067-105ec2bc4a34" />
<img width="1917" height="1026" alt="Airflow2" src="https://github.com/user-attachments/assets/cce4342f-490a-4199-831e-bc01e0f2d1de" />
<img width="1917" height="1025" alt="Airflow3" src="https://github.com/user-attachments/assets/9c44ff2a-5f21-425a-911b-f1528a2a4100" />



---

# 7️⃣ Snowflake

Snowflake is used as the cloud data warehouse.
<img width="1917" height="1031" alt="snowflake" src="https://github.com/user-attachments/assets/b81118f1-34c5-4fc8-9177-8281517bdf9e" />
<img width="1917" height="1020" alt="DIM_CUSTOMERS" src="https://github.com/user-attachments/assets/7b71539d-5b05-4c2f-8a94-fa116703b131" />



The pipeline uses Snowflake for:

- Raw data storage
- Staging transformations
- Analytics models
- Historical snapshots

### RAW Layer

Raw CDC data is loaded into the Snowflake RAW layer.

### ANALYTICS Layer

dbt creates analytics-ready models inside the ANALYTICS layer.

---

# 8️⃣ dbt Transformations

dbt is used to transform the raw Snowflake data into analytics-ready models.

### Staging Models

```text
stg_customers
stg_accounts
stg_transactions
```

### Dimension Models

```text
dim_customers
dim_accounts
```

### Fact Model

```text
fact_transactions
```

The final analytics models are stored in:

```text
BANKING.ANALYTICS
```

---

# 9️⃣ SCD Type-2 Implementation

The project demonstrates **Slowly Changing Dimension Type-2 (SCD2)** using dbt snapshots.

SCD2 preserves historical versions of records instead of simply overwriting the previous record.

### Example

Initial customer record:

```text
Customer ID: 3
Email: old_email@example.com
```

The customer record was then updated:

```text
Customer ID: 3
Email: scd2@example.com
```

The change travelled through the complete pipeline:

```text
PostgreSQL
    ↓
Debezium
    ↓
Kafka
    ↓
Python Consumer
    ↓
MinIO
    ↓
Airflow
    ↓
Snowflake RAW
    ↓
dbt
    ↓
dbt Snapshot
```

The SCD2 snapshot preserved both versions.

### SCD2 Result

```text
CUSTOMER_ID | EMAIL                  | DBT_VALID_FROM | DBT_VALID_TO
------------|------------------------|----------------|-------------
3           | old_email@example.com  | ...            | ...
3           | scd2@example.com       | ...            | NULL
```

The record where:

```text
DBT_VALID_TO IS NULL
```

represents the current version.

---

# 🧪 SCD2 Proof

The SCD2 implementation was validated using a real customer update.

### Step 1 – Initial Record

Customer ID `3` existed in the source system.

### Step 2 – Update

The customer's email was updated in PostgreSQL:

```text
scd2@example.com
```

### Step 3 – CDC

The update was captured by Debezium and published to Kafka.

### Step 4 – Consumer

The Python consumer consumed the Kafka event and uploaded the updated record to MinIO.

### Step 5 – Airflow

Airflow loaded the updated MinIO data into Snowflake RAW.

### Step 6 – dbt

The SCD2 DAG executed:

```text
dbt run
   ↓
dbt snapshot
```

### Step 7 – Historical Record

The Snowflake snapshot table contained:

```text
Old Version
    ↓
DBT_VALID_TO = timestamp

New Version
    ↓
DBT_VALID_TO = NULL
```

This confirms that the SCD Type-2 history was successfully maintained.

### 📸 SCD2 Proof

<img width="1917" height="1025" alt="Airflow3" src="https://github.com/user-attachments/assets/39aab907-f98e-420f-b631-7e8302092559" />
<img width="1917" height="1026" alt="SCD2 Check" src="https://github.com/user-attachments/assets/42f9cdbe-2030-435f-b1d6-62c7fd4b9d96" />


---

# 🔄 Complete End-to-End Pipeline

```text
                 ┌──────────────────────┐
                 │ Python Faker         │
                 │ Data Generator       │
                 └──────────┬───────────┘
                            │
                            ▼
                 ┌──────────────────────┐
                 │ PostgreSQL           │
                 │ OLTP Source          │
                 └──────────┬───────────┘
                            │
                            ▼
                 ┌──────────────────────┐
                 │ Debezium             │
                 │ Change Data Capture  │
                 └──────────┬───────────┘
                            │
                            ▼
                 ┌──────────────────────┐
                 │ Apache Kafka         │
                 │ CDC Events           │
                 └──────────┬───────────┘
                            │
                            ▼
                 ┌──────────────────────┐
                 │ Python Consumer      │
                 └──────────┬───────────┘
                            │
                            ▼
                 ┌──────────────────────┐
                 │ MinIO                │
                 │ Raw Parquet Storage  │
                 └──────────┬───────────┘
                            │
                            ▼
                 ┌──────────────────────┐
                 │ Apache Airflow       │
                 │ Orchestration        │
                 └──────────┬───────────┘
                            │
                            ▼
                 ┌──────────────────────┐
                 │ Snowflake RAW        │
                 └──────────┬───────────┘
                            │
                            ▼
                 ┌──────────────────────┐
                 │ dbt                  │
                 │ Transformations      │
                 └──────────┬───────────┘
                            │
                  ┌─────────┼─────────┐
                  ▼         ▼         ▼
              Dimensions   Facts     SCD2
                  │         │         │
                  └─────────┼─────────┘
                            ▼
                 ┌──────────────────────┐
                 │ Snowflake ANALYTICS  │
                 └──────────┬───────────┘
                            │
                            ▼
                 ┌──────────────────────┐
                 │ Power BI             │
                 │ Banking Dashboard    │
                 └──────────────────────┘
```

---

# 📊 Power BI Dashboard

The final analytics models are connected to Power BI for visualization.

The dashboard provides insights into:

- Customers
- Accounts
- Transactions
- Transaction amounts
- Transaction types
- Account-level transaction amounts
- Banking KPIs

### 📸 Power BI Dashboard

<img width="1338" height="750" alt="dashboard" src="https://github.com/user-attachments/assets/d1a85bde-7dbe-4938-b2e1-74166c27ad51" />


---

# 📂 Repository Structure

## 📂 Project Structure

```text
banking-modern-datastack-main/
├── .github/
│   └── workflows/
│
├── banking_dbt/
│   ├── models/
│   │   ├── staging/
│   │   └── marts/
│   │       ├── dimensions/
│   │       └── facts/
│   ├── snapshots/
│   ├── seeds/
│   ├── macros/
│   ├── tests/
│   └── dbt_project.yml
│
├── consumer/
│   └── kafka_to_minio.py
│
├── data-generator/
│   └── faker_generator.py
│
├── docker/
│   ├── dags/
│   ├── logs/
│   ├── minio/
│   ├── plugins/
│   └── postgres/
│
├── kafka-debezium/
│   └── generate_and_post_connector.py
│
├── postgres/
│   └── schema.sql
│
├── powebi/
│   └── dashboard.pbix
│
├── screenshots/
│
├── docker-compose.yml
├── dockerfile-airflow.dockerfile
├── requirements.txt
└── README.md
```

---

# 🚀 Key Features

- Synthetic banking data generation using Python Faker
- PostgreSQL OLTP source database
- Real-time Change Data Capture using Debezium
- Kafka event streaming
- Python Kafka consumer
- Parquet-based raw storage in MinIO
- Apache Airflow orchestration
- Snowflake RAW and ANALYTICS layers
- dbt staging models
- dbt dimension models
- dbt fact models
- dbt SCD Type-2 snapshots
- Historical customer tracking
- Power BI analytics dashboard
- Docker-based infrastructure

---

# 📈 Final Outcome

This project demonstrates a complete modern banking data engineering pipeline:

```text
Generate
   ↓
PostgreSQL
   ↓
Debezium CDC
   ↓
Kafka
   ↓
Python Consumer
   ↓
MinIO
   ↓
Airflow
   ↓
Snowflake RAW
   ↓
dbt
   ↓
Snowflake ANALYTICS
   ↓
SCD Type-2 History
   ↓
Power BI
```

The project demonstrates how modern data engineering tools can be combined to build an end-to-end banking analytics platform with streaming CDC, cloud data warehousing, orchestration, transformation, historical tracking, and business intelligence.

---

## 👨‍💻 Author

**Usama Patel**
