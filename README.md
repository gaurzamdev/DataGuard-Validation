# DataGuard — Data Validation & Reconciliation

A Python and PostgreSQL pipeline that validates incoming data, checks consistency across related datasets, and prevents discrepancies from going unnoticed.

### What it does

- Loads and standardizes raw CSV data.
- Runs data quality checks and removes duplicates.
- Detects missing or unmatched records.
- Compares daily record counts and revenue totals.
- Fails on unresolved discrepancies and stores diagnostic results.
- Tracks execution history and supports controlled, expiring exceptions.

## Architecture

| Schema | Responsibility |
|---|---|
| `raw` | Original source data |
| `staging` | Cleaned and typed records |
| `marts` | Reporting-ready datasets |
| `quality` | Validation results, reconciliation findings, and run history |

**Pipeline:** CSV ingestion → transformation → quality checks → reconciliation → daily revenue output → audit logging.

## Technology Stack

- Python 3.10+
- PostgreSQL
- Docker Compose
- GitHub Actions

## Quickstart

Start PostgreSQL:

```bash
docker compose up -d
```

Create a virtual environment and install dependencies:

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

**Run with consistent data:**

```bash
DATA_DIR=data_good python -m src.run_pipeline
```

Expected: successful execution when all checks pass.

**Run with inconsistent data:**

```bash
DATA_DIR=data python -m src.run_pipeline
```

Expected: failure when unresolved row-level mismatches or metric discrepancies are detected.

## Example Failure

The demonstration dataset contains a payment referencing a nonexistent order (`P9004` → `O9999`). The pipeline detects the inconsistency, records reconciliation findings, and rejects the run under strict mode.

Results are stored in:

- `quality.test_results`
- `quality.recon_row_mismatches`
- `quality.recon_summary`
- `quality.pipeline_runs`

## Audit & Exceptions

Every run is tracked using a `run_id`, with execution status, validation flags, mismatch counts, and timestamps.

Known discrepancies can be managed through `quality.recon_exceptions` using a tracking ticket and expiration date. Suppressed findings remain auditable.

## Continuous Integration

GitHub Actions verifies both scenarios:

- Consistent dataset → expected pass.
- Inconsistent dataset → expected pipeline failure.

## Key Engineering Concepts

- Data normalization and deduplication
- Referential integrity checks
- Row-level and aggregate reconciliation
- Repeatable pipeline execution
- Auditable data quality controls
- CI-based validation gates

## Future Improvements

- dbt for transformation management
- Airflow or Prefect for orchestration
- Data freshness checks and late-arriving record handling

## License

MIT