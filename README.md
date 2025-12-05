# 🚀 Real-Time Banking Analytics Platform with Kafka, Debezium, Snowflake, dbt & Airflow

_A Complete End-to-End Modern Data Engineering Project_

## 📘 Overview

This project builds a fully-functional **real-time banking data platform** using a cloud-native modern data stack.

It simulates real banking operations (customers, accounts, transactions), streams real-time database changes using **Kafka + Debezium**, lands raw data into **MinIO**, orchestrates pipelines with **Apache Airflow**, transforms data with **dbt**, stores analytics models in **Snowflake**, and finally visualizes insights in **Power BI**.

The entire workflow is automated using **CI/CD with GitHub Actions**.

👉 **This is a production-grade project** that mirrors how real financial institutions build scalable data ecosystems.

## 🏗️ System Architecture
![img_22.png](images%2Fimg_22.png)

## ⚡ Technology Stack
| Layer                          | Tools                                 |
| ------------------------------ | ------------------------------------- |
| **OLTP / Source System**       | PostgreSQL                            |
| **CDC Streaming**              | Kafka, Debezium                       |
| **Object Storage (Data Lake)** | MinIO                                 |
| **ETL/ELT Orchestration**      | Apache Airflow                        |
| **Cloud Data Warehouse**       | Snowflake                             |
| **Transformations & Modeling** | dbt (staging, marts, SCD-2 snapshots) |
| **Dashboarding**               | Power BI                              |
| **Automation**                 | GitHub Actions CI/CD                  |
| **Infrastructure**             | Docker & docker-compose               |
| **Data Simulation**            | Python + Faker                        |

## 🎯 Key Capabilities

- Real-time streaming from OLTP database → Data Lake → Snowflake

- End-to-end CDC via **Debezium** (reading WAL logs)

- Automated ingestion pipelines orchestrated using **Airflow**

- Clean and modeled data marts in **dbt**

- Slowly Changing Dimensions **(SCD Type-2)** using **dbt snapshots**

- CI/CD pipelines for dbt (testing, validation, deployment)

- Enterprise-level BI dashboards powered by **Power BI**

- Infrastructure fully containerized using **Docker**

## 📂 Repository Structure

```bash
    banking-modern-data/
├── .github/workflows/           # CI/CD pipelines (ci.yml, cd.yml)
│   ├── ci.yml                   # Runs dbt tests + linting
│   └── cd.yml                   # Deploys dbt models to Snowflake
│
├── banking_dbt/                 # dbt project (transforms, marts, snapshots)
│   ├── models/
│   │   ├── staging/             # Staging models (Bronze → Silver)
│   │   ├── marts/               # Facts & Dimensions (Gold layer)
│   │   └── sources.yml          # Source definitions
│   ├── snapshots/               # SCD Type-2 history tracking
│   └── dbt_project.yml
│
├── consumer/
│   └── kafka_to_minio.py        # CDC consumer → MinIO writer
│
├── data-generator/              # Synthetic banking dataset generator
│   └── faker_generator.py
│
├── docker/
│   ├── dags/                    # Airflow DAGs (Snowflake ingestion + snapshots)
│   ├── plugins/                 # Airflow plugins (if any)
│   └── minio/…                  # MinIO volume structure
│
├── kafka-debezium/
│   └── generate_and_post_connector.py
│
├── postgres/
│   └── schema.sql               # DDL + seed data for OLTP database
│
├── docker-compose.yml           # Infrastructure (Kafka, Debezium, Airflow, MinIO…)
├── dockerfile-airflow.dockerfile
├── requirements.txt
└── README.md

```
## 🧬 Step-by-Step Implementation
### 1️⃣ Data Simulation — Realistic Banking Operations

We generate **customers, accounts, and transactions** using Python + Faker.  
The data behaves like a real banking OLTP system with constraints, foreign keys, balances, and transactional behavior.

### ✔ Features
- Customer onboarding  
- Account opening  
- Deposits, withdrawals, transfers  
- Fraud-like transaction patterns  
- Inserted directly into **PostgreSQL OLTP**  
- 
![img_2.png](images%2Fimg_2.png)

### ▶️ Run the data generator:
```bash
python data-generator/faker_generator.py
```
![img_3.png](images%2Fimg_3.png)

**In Postgres:**
![img_4.png](images%2Fimg_4.png)

### 2️⃣ Real-Time CDC with Kafka + Debezium

Debezium monitors **PostgreSQL WAL logs**
→ captures **INSERT / UPDATE / DELETE**
→ streams changes into **Kafka topics**

Your connector automatically writes raw messages into MinIO through a consumer.


#### ▶️ Run the Debezium Connector Generator:
```bash 
python kafka-debezium/generate_and_post_connector.py
```

**Kafka topics created:**

- banking_server.public.customers

- banking_server.public.accounts

- banking_server.public.transactions

#### ⚠️ If Kafka libs fail, fix with:
```bash 
pip install --upgrade six
pip install --upgrade kafka-python
```
### 3️⃣ MinIO — S3-Compatible Data Lake (Bronze Layer)

Kafka messages are consumed and stored as Parquet files in MinIO.

#### ▶️ Run the Kafka → MinIO consumer:
```bash
python consumer/kafka_to_minio.py
```

![img_7.png](images%2Fimg_7.png)

#### ▶️ Required parquet package:
```bash
pip install fastparquet
```
![img_8.png](images%2Fimg_8.png)
![img_9.png](images%2Fimg_9.png)

### 4️⃣ Apache Airflow — The Orchestration Layer

Airflow automates the entire pipeline:

- Load raw data from MinIO → Snowflake (Bronze)

- Execute dbt models (Silver & Gold)

- Execute dbt snapshots (SCD Type-2)

- Daily scheduling or near real-time orchestration

Airflow DAGs live in:
```bash
docker/dags/
```
![img_10.png](images%2Fimg_10.png)


### 5️⃣ Snowflake — Cloud Data Warehouse

Snowflake stores analytics data across multiple processing layers:

| Layer      | Description                                 |
| ---------- | ------------------------------------------- |
| **Bronze** | Raw CDC data loaded from MinIO              |
| **Silver** | Cleaned & standardized staging models       |
| **Gold**   | Fact tables, dimensions, and business marts |

<!-- Snowflake DB screenshot -->

![img_11.png](images%2Fimg_11.png)

#### ▶️ Setup used:

- Warehouse: COMPUTE_WH

- Role: ACCOUNTADMIN

- Default DB: BANKING

- Schema: ANALYTICS

### 6️⃣ dbt Models & Transformations

dbt is used for transformations, tests, and snapshots.

#### ▶️ Install dbt:
```bash
pip install dbt-core
pip install dbt-snowflake
```


#### ▶️ Initialize the dbt project:
```bash
dbt init banking_dbt
```
Values you configured:
```bash
...
role: accountadmin
warehouse: COMPUTE_WH
database: banking
schema: analytics
threads: 4
```
#### ▶️ Test Snowflake connection:
```bash
dbt debug
```

##### ✔ dbt Staging Models

Clean and standardize raw CDC data.
![img_15.png](images%2Fimg_15.png)

##### ✔ dbt Marts

- dim_customers

- dim_accounts

- fact_transactions
- 
![img_17.png](images%2Fimg_17.png)

##### ✔ dbt Snapshots (SCD2)

Track history for:

- Customer attributes

- Account attributes

![img_16.png](images%2Fimg_16.png)

#### ▶️ Run all dbt models:
```bash
dbt run
```


#### ▶️ Run SCD Type-2 snapshots:
```bash
dbt snapshot
```

#### ▶️ Run only marts:
```bash
dbt run --select marts
```
### 7️⃣ CI/CD with GitHub Actions

Two workflow pipelines: CI and CD.


#### 🟦 CI Pipeline (ci.yml)

Triggers on:

- Push to dev

- Pull Request to main

CI performs:

- Setup Python environment

- Install dbt

- Install ruff

- Run dbt compile

- Run dbt test

- Validate code quality before merging

![img_18.png](images%2Fimg_18.png)

#### 🟩 CD Pipeline (cd.yml)

Triggers on:

- Merge PR from dev → main

CD performs:

- Run dbt models on Snowflake production

- Run dbt snapshots

- Run dbt tests

- Deploy transformations automatically

![img_19.png](images%2Fimg_19.png)

### 8️⃣ Power BI Dashboard — Real-Time Banking Analytics

Power BI connects directly to Snowflake to visualize:

#### 📊 Insights included:

- Customer growth trends

- Account activity over time

- Transaction insights (deposit / withdraw / transfer)

- Fraud-like anomalies

- SCD2 historical dimension tracking

![img_21.png](images%2Fimg_21.png)

---
## 📈 Final Outcomes

By the end of this project, you will have:

✔ A full data engineering pipeline running exactly like real banks
✔ Real-time CDC from PostgreSQL → Kafka → MinIO → Snowflake
✔ dbt + Snowflake star schema & SCD2 modeling
✔ Automated CI/CD for dbt
✔ Airflow orchestrating ingestion & transformations
✔ A complete Power BI dashboard
✔ 100% containerized, reproducible environment

---
## 👨‍💻 Author

**Diu Nguyen**

Data Engineer | Full Stack Developer

📧 nguyenhuongdiu1710@gmail.com