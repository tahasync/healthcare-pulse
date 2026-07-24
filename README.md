# Healthcare Pulse — Disease Surveillance & Forecast Dashboard

A Dockerized multi-service disease surveillance pipeline that scrapes public health data (WHO GHO API, OWID, CDC), processes it via Apache Spark with a GBTRegressor forecast model, and visualizes trends and regional breakdowns through a React dashboard — orchestrated with n8n and containerized via Docker Compose.

## What it does

Six Docker services (PostgreSQL, Flask API, React frontend, Spark master/worker, n8n) form a pipeline: scrapers pull disease data from WHO/OWID/CDC into raw CSVs; KNIME (running on the Windows host, not in Docker) performs ETL; PySpark cleans and transforms the data into a star-schema warehouse; a GBTRegressor model with 4 lag features generates forecasts; a Flask API with 15 endpoints serves metrics, cases, diseases, regions, and forecasts to a React dashboard with KPI cards, pie charts, trend lines, and a regional heatmap.

**Note:** KNIME runs on the Windows host (bridged via HTTP) — it is not containerized. The forecast endpoint falls back to random mock data when Parquet files are unavailable. The reported R² = 0.77 for the GBTRegressor is stated in the README but no evaluation script or cross-validation is committed to verify it.

## Tech stack

- **Backend:** Python 3.11, Flask 3.0, Gunicorn, psycopg2, pandas
- **Frontend:** React 18, Vite 5, Tailwind CSS 3, Recharts 2, Lucide React, Axios
- **Processing:** Apache Spark 3.4 (Bitnami Docker images), PySpark MLlib GBTRegressor
- **Database:** PostgreSQL 16 (star schema, 405 fact rows, 5 diseases, 180 regions)
- **ETL:** KNIME Analytics Platform (Windows host, HTTP bridge)
- **Orchestration:** n8n, Docker Compose (6 services), Nginx reverse proxy
- **Scraping:** WHO GHO API, Our World in Data CSV, CDC Open Data

## Features

- **Multi-source ingestion** — WHO GHO API, OWID Scrapy spider, CDC extractor
- **Spark ETL** — Deduplication, imputation, type casting via PySpark
- **GBTRegressor forecast** — 4 lag features, 2024–2025 case projections (with mock fallback)
- **Flask REST API** — 15 endpoints: metrics, cases, diseases, regions, forecast, pipeline status, KNIME trigger
- **React dashboard** — 5 pages: Dashboard (KPI grid + disease heatmap), Trends (forecast charts), RegionalMap (top-30 regions), Explorer (filterable data table), PipelineMonitor (stage states)
- **n8n orchestration** — 6-stage pipeline (scrape → ETL → clean → warehouse → ML → dashboard)
- **Docker Compose** — Single `docker compose up` starts all 6 services

## Setup

```bash
git clone https://github.com/tahasync/healthcare-pulse.git
cd healthcare-pulse
cp .env.example .env
# Edit .env with your passwords
docker compose up --build -d
# Dashboard: http://localhost, API: http://localhost:5000/api/health
```

For KNIME ETL: run `run_knime_bridge.bat` on the Windows host (requires KNIME Analytics Platform 5.1+).

## Status

**Academic project — functional demo.** All services build and run via Docker Compose. The Flask API returns real warehouse queries. The dashboard renders with live data when the pipeline has been run. Limitations: KNIME is not containerized (requires Windows host); the forecast model's R² claim cannot be reproduced from committed artifacts; the `/api/forecast` endpoint returns mock data when Parquet forecasts are absent; no automated tests exist.

**Team:** Muhammad Taha Naeem · Abdur Rehman · Adil Hayat Khan  
University of Central Punjab — Tools & Techniques for Data Science — Fall 2026