# Social Pulse Analytics Pipeline

An end-to-end data engineering pipeline that ingests, transforms, and serves social media engagement data using modern DE tools.

## Architecture

## Tech Stack

| Layer | Tool |
|-------|------|
| Ingestion | Python, Pandas |
| Storage | Snowflake |
| Transformation | dbt (SQL, Window Functions, CTEs) |
| Orchestration | Apache Airflow |
| Containerization | Docker |
| Version Control | GitHub |

## Pipeline Overview

### 1. Ingestion
- Loads 5000 rows of social media data from CSV into Snowflake RAW schema
- Uses `snowflake-connector-python` for bulk insert
- Truncate + reload strategy to prevent duplicates

### 2. Transformation (dbt Models)

**Staging:**
- `stg_social_media_posts` — cleans raw data, standardizes columns, calculates `engagement_rate_pct` and `total_interactions`

**Facts:**
- `fct_platform_performance` — ranks platforms by views and engagement using `RANK()` window functions
- `fct_top_posts` — ranks posts globally and within each platform using `PARTITION BY`
- `fct_engagement_trends` — calculates month over month growth using `LAG()` window function

### 3. Data Quality
- dbt tests: `unique`, `not_null`, `accepted_values`
- All 6 tests passing

### 4. Orchestration
- Airflow DAG runs daily
- Tasks: `ingest_csv_to_snowflake` → `dbt_run` → `dbt_test`
- Containerized with Docker

## Key SQL Concepts Used
- Window functions: `RANK()`, `LAG()`, `SUM() OVER()`
- CTEs for modular, readable SQL
- `PARTITION BY` for within-group rankings
- `NULLIF` for safe division
- `DATE_TRUNC` for time-based aggregations

## Results & Insights
- TikTok has highest engagement rate (63.87%)
- YouTube has most total views (3.37B)
- Top performing content: #viral shorts on TikTok in USA
- Month over month view growth tracked per platform and content type

## Project Structure
social_pulse_dbt/
├── models/
│   ├── staging/
│   │   ├── stg_social_media_posts.sql
│   │   └── schema.yml
│   └── marts/
│       ├── fct_platform_performance.sql
│       ├── fct_top_posts.sql
│       └── fct_engagement_trends.sql
├── dags/
│   └── social_pulse_dag.py
└── dbt_project.yml


## How to Run

1. Set up `.env` with Snowflake credentials
2. Run ingestion: `python ingestion/reddit_ingest.py`
3. Run dbt: `dbt run && dbt test`
4. Or trigger Airflow DAG: `social_pulse_pipeline`

