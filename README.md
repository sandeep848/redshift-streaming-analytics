# Redshift Streaming Analytics

[![Python](https://img.shields.io/badge/Python-3.11%2B-3776AB.svg)](https://www.python.org/)
[![Kafka](https://img.shields.io/badge/Streaming-Kafka-231F20.svg)](https://kafka.apache.org/)
[![Docker](https://img.shields.io/badge/Runtime-Docker-2496ED.svg)](https://www.docker.com/)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

An end-to-end streaming analytics pipeline for replaying, validating and analyzing Amazon Redshift query metrics. Kafka carries the event stream, DuckDB provides a lightweight local analytics store, and Streamlit exposes operational and historical views.

## Architecture

~~~mermaid
flowchart LR
    A["Query metrics"] --> B["Kafka producer"]
    B --> C["Kafka topics"]
    C --> D["DuckDB consumer"]
    C --> E["Redshift loader"]
    D --> F["Streamlit dashboard"]
    E --> G["S3 and Redshift"]
~~~

## Features

- Cleaning, enrichment and deterministic de-duplication
- Kafka raw and processed topics
- DuckDB ingestion and analytical rollups
- Optional S3 staging and Redshift `COPY` loading
- Typed configuration with environment overrides
- Structured logging and checkpointed replay state
- Docker Compose development environment
- Automated unit tests and end-to-end sanity checks

## Quick start

~~~bash
git clone https://github.com/sandeep848/Redshift_Analytics.git
cd Redshift_Analytics
python -m venv .venv
source .venv/bin/activate
pip install -e ".[dev]"
cp .env.example .env
docker compose up -d
make topics
make bootstrap
~~~

On Windows, use `.venv\Scripts\Activate.ps1` and copy the environment template with `copy .env.example .env`.

Run the components in separate terminals:

~~~bash
python -m src.main consumer-duckdb
python -m src.main producer
python -m src.main ui
~~~

## Command-line interface

~~~bash
python -m src.main --help
~~~

| Command | Purpose |
|---|---|
| `producer` | Replay query metrics into Kafka |
| `consumer-duckdb` | Persist events and update DuckDB rollups |
| `consumer-redshift` | Stage batches to S3 and load Redshift |
| `bootstrap-duckdb` | Initialize the local analytics schema |
| `ui` | Start the Streamlit application |

## Testing

~~~bash
pytest
~~~

Run the local end-to-end checks with:

~~~bash
python -m scripts.run_sanity_checks
~~~

## Configuration and security

Configuration defaults live in `configs/app.yaml` and can be overridden through environment variables documented in [`.env.example`](.env.example). Real AWS credentials and application secrets must never be committed. For production, use an external secret manager and least-privilege IAM roles.

## Repository structure

~~~text
configs/            # Application, logging and SQL configuration
src/common/         # Settings, schemas and logging
src/producer/       # Kafka replay producer
src/consumers/      # DuckDB and Redshift consumers
src/storage/        # DuckDB, S3 and Redshift clients
src/ui/             # Streamlit interface
tests/              # Automated tests
docs/               # Architecture and configuration notes
~~~

## Deployment

~~~bash
docker build -t redshift-streaming-analytics .
~~~

The optional Redshift path requires an AWS account, S3 bucket, Redshift cluster and correctly scoped permissions. The DuckDB path can be run entirely locally.

## Limitations

- The repository demonstrates the data path but does not provision managed AWS infrastructure.
- Streamlit live views require the local pipeline to be running.
- Production deployments still need managed observability, secret rotation and retention policies.

## Documentation

- [Architecture](docs/architecture.md)
- [Configuration](docs/configuration.md)
- [Kafka contracts](docs/kafka.md)
- [API notes](docs/api.md)

## License

Distributed under the [MIT License](LICENSE).
