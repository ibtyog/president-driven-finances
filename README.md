# PDF (President Driven Finances)

Event-study BI pipeline analyzing Donald Trump's Truth Social activity and cryptocurrency market reactions.

## Repository Layout

- `ingestion/` — Part 1 data collection (Truth Social archive, Binance klines & Mempool transaction data → Bronze Parquet)
- `etl/` — Skeleton for future DuckDB/dbt transformations, Great Expectations checks, and Dagster orchestration
- `ai/` — Skeleton reserved for 1–2 future NLP/ML modules
- `tests/` — Automated test suite
- `docs/` — Architecture decision records and implementation plans

## Local Usage (`uv`)

```bash
uv sync --extra ingestion
cp .env.example .env  # optional, edit to customize interval/symbols/storage

# Run once for all sources (or pick --source truth_social | binance)
uv run ingest run
uv run ingest run --source binance

# Run continuously on POLL_INTERVAL_SECONDS (default 300s)
uv run ingest loop
```

Parquet files land under `data/bronze/` and persistent rotating logs write to `data/logs/ingestion.log`.

### Switching Candle Interval (`1m` → `5m`)

Set `BINANCE_INTERVAL=5m` in `.env` (or pass `BINANCE_INTERVAL=5m uv run ingest run`). State watermarks are keyed per `(symbol, interval)`, so intervals do not collide.

## Docker Usage

```bash
docker compose up -d --build
docker compose logs -f ingestion
```

Persistent Bronze data and rotating logs stay mapped to `./data` on the host.

## Adding a Third Data Source

1. Create a new connector file in `ingestion/sources/` implementing `SourceConnector` (`ingestion/sources/base.py`).
2. Register it in `build_registry()` and the `--source` CLI choices in `ingestion/cli.py`.
