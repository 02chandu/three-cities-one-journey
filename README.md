# Three Cities, One Journey

## Azure Data Engineering – Multi-Source Bike Share Analytics Platform

### 1. Business Problem

Bike-share operators in different cities publish trip data using different schemas,
timestamp formats, station identifiers, and trip definitions.

This project builds a unified data platform that integrates:

- Citi Bike – New York
- Divvy – Chicago
- Open-Meteo – Weather data

The goal is to transform these incompatible source datasets into a common analytical
model so that the same business questions can be answered consistently across cities.

---

## 2. Project Objectives

The platform will:

- Ingest raw source data without modifying it.
- Preserve source-specific schemas in the Bronze layer.
- Conform different source datasets into a common Silver trip model.
- Handle schema changes using a versioned schema registry.
- Quarantine invalid records instead of silently dropping them.
- Apply data-quality gates between processing layers.
- Enrich trip data with hourly weather information.
- Build analytical Gold marts.
- Support reproducible full rebuilds.
- Provide Power BI dashboards for analysis.

---

## 3. Architecture

### High-Level Data Flow

Citi Bike ───────┐
                 │
Divvy ───────────┼──> ADF ──> ADLS Gen2 Bronze
                 │                    │
Open-Meteo ──────┘                    ↓
                              Azure Databricks
                              PySpark Processing
                                     │
                              Silver Layer
                                     │
                                     ↓
                                Gold Layer
                                     │
                                     ↓
                                  Power BI

### Medallion Architecture

Bronze
- Raw source data
- Immutable
- Source files preserved as published
- Ingestion metadata added

Silver
- Conformed trip model
- Standardized timestamps
- Standardized station identifiers
- Data-quality validation
- Invalid records quarantined
- Weather enrichment

Gold
- Business-ready analytical marts
- Station-hour demand
- Daily city comparison
- Weather sensitivity analysis

---

## 4. Technology Stack

| Area | Technology |
|---|---|
| Cloud | Microsoft Azure |
| Storage | Azure Data Lake Storage Gen2 |
| Ingestion | Azure Data Factory |
| Transformation | Azure Databricks |
| Processing | PySpark |
| Storage Format | Parquet / Delta |
| Data Quality | PySpark / SQL / dbt-style tests |
| Orchestration | Azure Data Factory |
| Visualization | Power BI |
| Version Control | Git / GitHub |

---

## 5. Data Sources

### Citi Bike

New York City bike-share trip data.

Source:
https://s3.amazonaws.com/tripdata/index.html

### Divvy

Chicago bike-share trip data.

Source:
https://divvy-tripdata.s3.amazonaws.com/index.html

### Open-Meteo

Historical weather data.

Source:
https://archive-api.open-meteo.com/v1/archive

---

## 6. Data Layers

### Bronze Layer

Purpose:

Preserve source data exactly as published.

Rules:

- Never modify source records.
- Store source files in their original form.
- Add source identifier.
- Add ingestion timestamp.
- Track source file and ingestion metadata.

Example:

```text
bronze/
├── citibike/
│   └── 2025/
├── divvy/
│   └── 2025/
└── weather/
    └── 2025/
