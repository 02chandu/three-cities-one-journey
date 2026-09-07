# Architecture Design

## 1. Architecture Overview

The Three Cities, One Journey platform follows a Medallion architecture
with three clearly separated processing layers:

Source Systems
      ↓
Azure Data Factory
      ↓
Bronze
      ↓
Azure Databricks / PySpark
      ↓
Silver
      ↓
Gold
      ↓
Power BI

## 2. Source Systems
2.1 Citi Bike

Source:

New York City bike-share trip data.

Characteristics:

Monthly files
Source schemas may change over time
Trip definitions and station information must be standardized
2.2 Divvy

Source:

Chicago bike-share trip data.

Characteristics:

Monthly files
Schema differs from Citi Bike
Station identifiers and trip attributes require conformance
2.3 Open-Meteo

Source:

Historical weather data.

Weather data will be used to enrich bike-share trips using the trip
start time and appropriate city/location.

## 3. Ingestion Layer – Azure Data Factory

Azure Data Factory is responsible for source ingestion and orchestration.

Responsibilities:

Identify source files/data.
Copy source data into ADLS Gen2.
Preserve source data.
Add ingestion metadata.
Orchestrate downstream processing.
Handle pipeline failures.
Pass configuration to processing activities.

ADF should not contain the main business transformation logic.

## 4. Storage – Azure Data Lake Storage Gen2

ADLS Gen2 provides the central storage layer.

The lake is organized according to the Medallion architecture.

ADLS Gen2
│
├── bronze/
│
├── silver/
│
├── gold/
│
└── quarantine/
5. Bronze Layer
Purpose

The Bronze layer stores source data in its original form.

Responsibilities
Preserve source files.
Maintain source lineage.
Store ingestion metadata.
Support replay and reprocessing.
Rules
Bronze data must not be modified.
Source data must remain traceable.
Source-specific structure is allowed.
Transformation logic does not belong in Bronze.

Example:

bronze/
├── citibike/
│   └── YYYY/
│       └── MM/
│
├── divvy/
│   └── YYYY/
│       └── MM/
│
└── weather/
    └── YYYY/
        └── MM/
## 6. Schema Registry

The project uses a versioned schema registry.

Purpose:

Handle source schema changes without spreading hardcoded
source-specific logic throughout the pipeline.

Example:

config/
└── schema_registry/
    ├── citibike/
    │   ├── schema_v1.json
    │   └── schema_v2.json
    │
    ├── divvy/
    │   └── schema_v1.json
    │
    └── weather/
        └── schema_v1.json

Each schema definition should identify:

Source
Period
Schema version
Columns
Data types
Required fields

## 7. Transformation Layer – Azure Databricks

Azure Databricks with PySpark performs the main transformation work.

Responsibilities:

Read Bronze data.
Apply schema definitions.
Standardize data types.
Normalize timestamps.
Standardize trip definitions.
Conform station information.
Apply data-quality rules.
Create Silver datasets.
Enrich data with weather.
Create Gold analytical marts.
8. Silver Layer
Purpose

Create a common analytical model across different source systems.

The Silver layer provides a conformed trip model.

Example conceptual model:

trip_id
source
city
started_at
ended_at
start_station_id
start_station_name
end_station_id
end_station_name
start_lat
start_lon
end_lat
end_lon
trip_duration
bike_type
weather attributes

The exact model will be finalized during implementation after examining
the source schemas.

## 9. Data Quality and Quarantine

Data quality checks are applied before publishing data to the next layer.

Example:

Bronze
   │
   ▼
Validation
   │
   ├──────────────► Valid Records
   │                     │
   │                     ▼
   │                  Silver
   │
   └──────────────► Invalid Records
                         │
                         ▼
                    Quarantine

Quarantined records should contain:

Source
Source file
Rejection reason
Processing timestamp

Critical quality failures should prevent affected output from being
published.

10. Weather Enrichment

Weather data will be aligned with trip activity.

The primary matching concept is:

Trip Start Time
      ↓
Timezone Normalization
      ↓
Nearest Hour
      ↓
Weather Observation

Timezone and daylight-saving behavior must be considered during
implementation.

## 11. Gold Layer

The Gold layer contains business-ready analytical datasets.

Gold Mart 1 – Station-Hour Demand

Purpose:

Analyze bike-share demand by station and hour.

Example dimensions:

City
Station
Date
Hour

Example measures:

Trip count
Average trip duration
Gold Mart 2 – Normalized Daily City Comparison

Purpose:

Compare bike-share activity across cities using normalized metrics.

Example dimensions:

City
Date

Example measures:

Daily trips
Normalized trips
Average trip duration
Gold Mart 3 – Weather Sensitivity

Purpose:

Analyze how weather conditions affect bike-share usage.

Example dimensions:

City
Date
Hour
Weather condition

Example measures:

Trip count
Temperature
Precipitation
Weather-related metrics

## 12. Power BI

Power BI consumes the Gold datasets.

The dashboard will provide:

Cross-city comparison
Station demand analysis
Hourly demand patterns
Weather impact
Key KPIs

Power BI should primarily consume business-ready Gold data rather than
performing the core transformation logic itself.

## 13. Processing Flow

The complete processing flow is:

Citi Bike ──────┐
                │
Divvy ──────────┼──► ADF
                │
Open-Meteo ─────┘
                   │
                   ▼
              ADLS Bronze
                   │
                   ▼
            Schema Validation
                   │
                   ▼
             Databricks
                   │
             ┌─────┴─────┐
             ▼           ▼
          Valid        Invalid
             │           │
             ▼           ▼
          Silver     Quarantine
             │
             ▼
       Weather Enrichment
             │
             ▼
           Gold
             │
             ▼
         Power BI
         
## 14. Design Principles
Immutable Bronze

Raw source data is preserved and is not modified.

Configuration Over Hardcoding

Source-specific schema and configuration should be externalized.

Quality Gates

Invalid data should not silently move into downstream layers.

Quarantine Instead of Drop

Invalid records should be retained with rejection reasons.

Idempotency

Repeated processing should not create duplicate output.

Observability

Pipeline processing and data-quality results should be measurable.

Reproducibility

The platform should be rebuildable from zero using a documented process.

## 15. Azure Service Responsibilities
Azure Service	Responsibility
Azure Data Factory	Ingestion and orchestration
ADLS Gen2	Data lake storage
Azure Databricks	Transformation and processing
Power BI	Analytics and visualization

The project intentionally keeps responsibilities separated so that
ingestion, storage, transformation, and visualization have clear
boundaries.
