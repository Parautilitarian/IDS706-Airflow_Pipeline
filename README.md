# Airflow ETL and Analysis Pipeline

## Overview
This project demonstrates a complete **end‑to‑end ETL and analysis pipeline** deployed with **Apache Airflow** running in Docker.  
It generates synthetic data with [Faker](https://faker.readthedocs.io/), merges multiple datasets, loads them into **PostgreSQL**, performs analysis and visualization with **pandas + matplotlib**, and finally cleans up intermediate artifacts.

---

## Deployment

### 1 Docker‑Compose Stack
Airflow is deployed with **CeleryExecutor**, **Postgres**, and **Redis** using `docker‑compose.yml`.

Services include:

- `airflow-scheduler`
- `airflow-worker`
- `airflow-dag-processor`
- `airflow-triggerer`
- `airflow-apiserver` (API/Web UI)
- `db` (PostgreSQL 16)
- `redis` (broker)

Extra Python libraries are installed automatically via `_PIP_ADDITIONAL_REQUIREMENTS`:
apache‑airflow‑providers‑postgres
matplotlib
pandas
faker
psycopg2‑binary


> During first startup, run  
> ```bash
> echo "AIRFLOW_UID=$(id -u)" > .env
> docker compose build --no-cache
> docker compose up -d
> ```
> then open **http://localhost:8080** (username `airflow`, password `airflow`).

---

## DAG: `pipeline.py`

### 2 Data Ingestion & Transformation
The DAG (`dags/pipeline.py`) is scheduled with `@once` and organized as a sequence of Python @task functions:

| Task | Description |
|------|--------------|
| **`fetch_persons()`** | Generate fake person records (first name, last name, email, address…). |
| **`fetch_companies()`** | Generate fake company profiles (name, industry, employees count etc.). |
| **`merge_csvs()`** | Combine the two CSVs by index to create a merged dataset linking people to companies. |
| **`load_csv_to_pg()`** | Create a schema (`week8_demo`) and table (`employees`) in PostgreSQL, then bulk‑insert merged rows. |

Each dataset is written to `/opt/airflow/data/*.csv` inside the container.

---

### 3 Analysis & Visualization
After successful loading into the database, the DAG executes:

| Task | Description |
|------|--------------|
| **`analyze_and_visualize()`** | Reads the merged table from Postgres using `PostgresHook`, converts to a **pandas** DataFrame, and produces a **matplotlib** bar chart. Example: *Top 10 companies by record count*. |
| **`clear_folder()`** | Deletes temporary CSV files in the data folder after analysis to keep containers clean. |

Dependencies:
(fetch_persons, fetch_companies)
>> merge_csvs
>> load_csv_to_pg
>> analyze_and_visualize
>> clear_folder


Result artifacts (e.g., `top_companies.png`) are stored in `/opt/airflow/data/`.

---

## Output Example
- `persons.csv`
- `companies.csv`
- `merged_data.csv`
- `top_companies.png` – visualization of the analysis result.

All intermediate files are removed by `clear_folder()` after successful execution.

---

## Technologies Used
| Category | Tools |
|-----------|-------|
| Workflow Orchestration | **Apache Airflow 2** |
| Containers | **Docker / Docker Compose** |
| Database | **PostgreSQL 16** |
| Message Broker | **Redis 7** |
| Data Generation | **Faker** |
| Data Analysis | **pandas**, **matplotlib** |
| Language | **Python 3.12** |

---

## Cleanup
`clear_folder()` automatically deletes intermediate CSV files to conserve disk space.  
To bring the whole stack down:

```bash
docker compose down -v
