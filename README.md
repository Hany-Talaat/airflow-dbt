# Analytics Engineering Workspace

This workspace contains a local DuckDB and dbt analytics pipeline, plus a separate Apache Airflow project scaffold.

## Projects

### `duckdb_dbt_etl/`

The main data pipeline. It loads CSV files from `data/raw/` into the `ods` schema of `dbt_project/dev.duckdb`, then uses dbt to build staging views and dimensional warehouse tables.

- `scripts/ods.py` loads the source CSV files into DuckDB.
- `dbt_project/` contains the dbt project and SQL models.
- `scripts/marts.py` prints row counts and data from the built warehouse tables.
- `logs/` contains dbt logs; `.venv/` is the local Python environment.

The source CSVs expected by the loader are `accounts.csv`, `data_dictionary.csv`, `products.csv`, `sales_pipeline.csv`, and `sales_teams.csv` under `duckdb_dbt_etl/data/raw/`.

The dbt models are organized into:

- **Staging:** source cleanup models such as `stg_accounts`, `stg_products`, `stg_sales_pipeline`, and `stg_sales_teams`.
- **Marts:** dimensions for accounts, dates, deal stages, products, sales agents, and sub-accounts, plus the `fact_sales` table.

### `../airflow/airflow_project/`

A separate Airflow project scaffold with its own Dockerfile, requirements, and Airflow configuration. Its `dags/` folder currently contains only `.airflowignore`; no pipeline DAG is defined there yet.

## Run the DuckDB and dbt pipeline

From `duckdb_dbt_etl/`, activate the existing virtual environment in PowerShell:

```powershell
.\.venv\Scripts\Activate.ps1
```

Load the CSVs, build the dbt models, and run dbt tests:

```powershell
python .\scripts\ods.py
cd .\dbt_project
dbt debug
dbt build
```

`dbt build` runs models and tests. The dbt profile must be configured for the local DuckDB database at `duckdb_dbt_etl/dbt_project/dev.duckdb`; `dbt debug` checks that configuration before building.

To print the resulting warehouse tables, return to `duckdb_dbt_etl/` and run:

```powershell
python .\scripts\marts.py
```

## Notes

- Run `ods.py` before dbt so the `ods` source tables exist.
- dbt staging models are configured as views in schema `stg`; marts are configured as tables in schema `dwh` (see `duckdb_dbt_etl/dbt_project/dbt_project.yml`).
- Keep local credentials, virtual environments, generated dbt output, and the DuckDB database out of source control where appropriate.
