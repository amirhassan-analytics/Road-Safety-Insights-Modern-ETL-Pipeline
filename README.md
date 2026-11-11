# Road Safety Insights — Modern ETL Pipeline

End-to-end data engineering pipeline for accident data analysis: ingestion, cleaning,
warehousing, transformation and reporting. A modern rebuild of the original
university project (`Road-Safety-Insights-ETL-Pipeline`), replacing the SSIS /
SSAS workflow with Python, PostgreSQL, Apache Airflow and dbt-style SQL models —
the same stack used in production data teams.

## What it does

| Stage | What happens |
|---|---|
| Extract | Reads the three source datasets — **Crashes**, **People**, **Vehicles** — from `data/raw/` (CSV) |
| Transform | Standardises column names, imputes missing values (mean/median for numeric, mode for categorical), deduplicates, derives features (e.g. day–night crash index) |
| Load | Loads cleaned data into a PostgreSQL star schema (`fact_crashes`, `dim_vehicle`, `dim_person`, `dim_date`, `dim_street`) |
| Quality | Runs data-quality checks (nulls, duplicates, referential integrity, value ranges) and writes a report to `reports/data_quality.json` — the pipeline fails if a check fails |
| Model | dbt-style SQL models in `models/` build analytics tables for Power BI |
| Report | Power BI dashboards: damage costs by geography, crashes by street attributes (lighting, visibility), people involved by role / injury / age |

## Architecture

```
data/raw/*.csv
      │  extract.py
      ▼
data/staging/ (cleaned parquet/csv)
      │  transform.py
      ▼
PostgreSQL (sql/schema.sql — star schema)
      │  load.py
      ▼
models/ (staging → marts, SQL)
      ▼
Power BI (.pbix in the original repo)
```

Orchestration: `dags/road_safety_etl_dag.py` (Apache Airflow) runs
extract → transform → load → data-quality, daily.

## Quick start

```bash
# 1. Put the three source CSVs in place (not included — see data/README.md)
cp crashes.csv people.csv vehicles.csv data/raw/

# 2. Start PostgreSQL
docker compose up -d postgres

# 3. Install dependencies
pip install -r requirements.txt

# 4. Run the whole pipeline
python -m src.run_pipeline

# 5. Check the quality report
cat reports/data_quality.json
```

Or run everything in Docker:

```bash
docker compose up --build
```

## Project structure

```
├── README.md
├── docker-compose.yml      # PostgreSQL (+ Airflow-ready)
├── Dockerfile
├── requirements.txt
├── .env.example
├── src/
│   ├── config.py           # paths, DB connection, settings
│   ├── extract.py          # read raw CSVs → staging
│   ├── transform.py        # clean, impute, derive features
│   ├── load.py             # load into PostgreSQL star schema
│   ├── data_quality.py     # checks + JSON report
│   └── run_pipeline.py     # extract → transform → load → quality
├── sql/
│   └── schema.sql          # star schema DDL
├── models/
│   ├── staging_crashes.sql
│   ├── mart_crash_summary.sql
│   └── mart_people_injuries.sql
├── dags/
│   └── road_safety_etl_dag.py
├── tests/
│   └── test_data_quality.py
├── data/
│   └── README.md           # where the source CSVs go
└── .github/workflows/ci.yml
```

## How this maps to my experience

This is the project behind the **Incident Data Warehouse and ETL Pipeline**
on my CV (Team Lead, Dec 2025 – August 2026, Self based). The rebuild
adds the production practices from my TechWelt Group work: scheduled
orchestration, data-quality checks on every pipeline, and a documented,
reproducible setup another engineer can run with one command.

## Tech

Python · SQL · PostgreSQL · Apache Airflow · dbt-style SQL · Docker ·
pandas · SQLAlchemy · Power BI

## Notes

- The original SSIS packages, MDX cube queries and Power BI `.pbix` from the
  university project are kept in the original repo history; this rebuild
  replaces the SSIS step with Python so the pipeline runs on any OS.
- Column names in your CSVs may differ from the ones assumed here — adjust
  the mapping in `src/config.py` (`COLUMN_MAP`) to match your files.
