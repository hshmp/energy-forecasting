# Energy Demand Forecasting (Victoria, Australia)

![Databricks](https://img.shields.io/badge/Databricks-FF3621?style=for-the-badge&logo=databricks&logoColor=white)
![PySpark](https://img.shields.io/badge/PySpark-E25A1C?style=for-the-badge&logo=apachespark&logoColor=white)
![Power BI](https://img.shields.io/badge/Power%20BI-F2C811?style=for-the-badge&logo=powerbi&logoColor=black)
![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white)

An end-to-end energy demand forecasting pipeline that ingests 5-minute interval data from the **Australian Energy Market Operator (AEMO)** public dataset, processes it through a **medallion architecture** on Databricks, and produces hourly demand forecasts for the Victoria (VIC1) region using **XGBoost** which is then visualised in Power BI.

## Dashboard

![Dashboard Preview](reports/dashboard_preview.png)

## Architecture

```mermaid
flowchart TD
    A([🌐 AEMO Public API\nNEM Price & Demand CSVs]) --> B

    B["🥉 BRONZE\nRaw CSV files — one per month\n/Volumes/workspace/default/bronze/"]
    B -->|parse · deduplicate\nfilter TRADE intervals| C

    C["🥈 SILVER\nCleaned Delta table\nworkspace.default.energy_clean"]
    C -->|hourly aggregation\nfeature engineering| D

    D["🥇 GOLD\nML-ready features + forecast output\nworkspace.default.demand_hourly"]
    D -->|XGBoost · lag features\ncyclic encoding| E

    E["📊 Power BI Dashboard\nActual vs Predicted demand\nreports/energy_forecast.pbix"]

    style A fill:#222,color:#fff,stroke:none
    style B fill:#222,color:#fff,stroke:none
    style C fill:#222,color:#fff,stroke:none
    style D fill:#222,color:#fff,stroke:none
    style E fill:#222,color:#fff,stroke:none
```

---

## Results

<!-- TODO: Run 4_forecasting.ipynb and fill in the metrics printed to the cell output -->
<!-- Example of what a filled-in row looks like:  MAE | 182.4 MW -->

| Metric | Value |
|--------|-------|
| Forecast target | Hourly average demand (MW) — VIC1 |
| Training period | 2020 – 2023 |
| Test period | 2024 (full year holdout, ~8,760 hours) |
| MAE | 353.54 mw |
| RMSE | 495.51 mw |
| MAPE | 8.09% |

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Platform | Databricks |
| Processing | PySpark |
| ML | XGBoost, scikit-learn |
| Visualisation | Power BI |
| Data source | [AEMO NEM Price and Demand](https://www.aemo.com.au/energy-systems/electricity/national-electricity-market-nem/data-nem/aggregated-data) |

---

## Notebooks

Run in order:

| # | Notebook | Description |
|---|----------|-------------|
| 0 | `0_config.ipynb` | Creates Bronze / Silver / Gold Unity Catalog volumes |
| 1 | `1_bronze_ingest.ipynb` | Downloads monthly AEMO CSVs (2020 → present) into Bronze volume |
| 2 | `2_silver_transform.ipynb` | Cleans, deduplicates, and upserts to Silver Delta table |
| 3 | `3_gold_aggregate.ipynb` | Aggregates to hourly buckets and engineers time features |
| 4 | `4_forecasting.ipynb` | Trains XGBoost model and writes predictions to Gold forecast table |
| 5 | `5_export_pbi.dbquery.ipynb` | Exports forecast table for Power BI consumption |

---

## Setup

### Prerequisites

- Databricks workspace with Unity Catalog enabled
- Cluster running **Databricks Runtime 14+** (Python 3.10+)
- `xgboost` installed on the cluster (or run `%pip install xgboost` in notebook 4)

---

## Feature Engineering

The XGBoost model uses the following features:

| Feature | Description |
|---------|-------------|
| `hour_sin`, `hour_cos` | Cyclic encoding of hour-of-day |
| `month_sin`, `month_cos` | Cyclic encoding of month |
| `dow_sin`, `dow_cos` | Cyclic encoding of day-of-week |
| `lag_day` | Demand 24 hours prior |
| `lag_week` | Demand 168 hours (1 week) prior |
| `rolling_week_avg` | 7-day rolling average demand |
| `is_weekend` | Binary weekend indicator |

---

## Data Source

Data is sourced from AEMO's public NEM aggregated data files:

```
https://www.aemo.com.au/aemo/data/nem/priceanddemand/PRICE_AND_DEMAND_{YEAR}{MONTH}_{REGION}.csv
```

- **Region:** VIC1 (Victoria)
- **Interval:** 5 minutes
- **Filter:** `PERIODTYPE = 'TRADE'` intervals only

No API key or authentication required. Data is freely available under AEMO's public data licence.

---

## Project Structure

```
energy-forecasting/
├── 0_config.ipynb               # volume setup and constants
├── 1_bronze_ingest.ipynb        # data ingestion
├── 2_silver_transform.ipynb     # cleaning
├── 3_gold_aggregate.ipynb       # feature engineering
├── 4_forecasting.ipynb          # XGBoost forecasting model
├── 5_export_pbi.dbquery.ipynb   # Power BI export SQL
├── reports/
│   ├── energy_forecast.pbix     # Power BI report
│   ├── dashboard_preview.png    # screenshot
│   └── energy_forecast.pdf      # PDF version
└── README.md
```
