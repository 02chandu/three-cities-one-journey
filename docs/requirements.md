# Project Requirements

## 1. Project Name

Three Cities, One Journey

---

## 2. Business Requirement

Build a data platform that combines bike-share trip data from different
city operators and makes the data comparable through a common analytical
model.

The platform must support consistent analysis across:

- New York – Citi Bike
- Chicago – Divvy

Weather data from Open-Meteo will be used to enrich trip activity.

---

# 3. Functional Requirements

## FR-01 – Source Data Ingestion

The platform must ingest data from:

- Citi Bike
- Divvy
- Open-Meteo

The ingestion process must preserve the original source data.

---

## FR-02 – Bronze Layer

The Bronze layer must:

- Store source files as published.
- Preserve the original source structure.
- Add source identifier.
- Add ingestion timestamp.
- Store source/file metadata where required.
- Never modify previously ingested Bronze data.

---

## FR-03 – Schema Management

The platform must maintain a versioned schema registry.

The registry must identify:

- Source
- Data period
- Schema version
- Expected columns
- Expected data types

A new source schema should be handled through the schema registry
rather than by changing the core transformation logic.

---

## FR-04 – Silver Conformed Model

The Silver layer must create one common trip model across the different
bike-share operators.

The model must standardize:

- Column names
- Data types
- Timestamps
- Trip definitions
- Station identifiers

---

## FR-05 – Data Quality

Data quality checks must be performed between processing layers.

Examples include:

- Required fields
- Valid timestamps
- Valid trip duration
- Valid station information
- Valid coordinates
- Duplicate detection
- Schema validation

---

## FR-06 – Quarantine

Invalid records must not simply be dropped.

They must be moved to a quarantine area with information such as:

- Rejection reason
- Source
- Source file
- Processing timestamp

The quarantine rate must be measurable.

---

## FR-07 – Quality Gates

A quality gate must exist between processing layers.

If critical quality checks fail:

- The affected output must not be published.
- The pipeline must report the failure.
- The failure must be observable.

---

## FR-08 – Weather Enrichment

Trip data must be enriched with hourly weather information.

Weather matching must use:

- Trip start time
- Appropriate city/location
- Nearest hourly weather observation

Timezone and daylight-saving considerations must be handled.

---

## FR-09 – Gold Data Marts

The platform must create at least three analytical Gold marts:

### Gold Mart 1 – Station-Hour Demand

Analyze bike-share demand by:

- City
- Station
- Hour

### Gold Mart 2 – Normalized Daily City Comparison

Compare cities using normalized metrics rather than relying only
on raw trip counts.

### Gold Mart 3 – Weather Sensitivity

Analyze the relationship between weather conditions and bike-share usage.

---

## FR-10 – Reproducibility

The complete platform must be reproducible.

A documented process must allow the project to be rebuilt from zero.

---

## FR-11 – Idempotency

Repeated execution of the same pipeline operation must not create
duplicate records or duplicate output.

---

# 4. Non-Functional Requirements

## NFR-01 – Reliability

The pipeline should detect failures and prevent invalid data from
being published.

---

## NFR-02 – Data Integrity

Bronze data must remain unchanged after ingestion.

---

## NFR-03 – Maintainability

Business logic and source-specific configuration should be separated
where possible.

---

## NFR-04 – Scalability

The design should allow the processing approach to scale beyond
laptop-sized datasets.

---

## NFR-05 – Observability

The platform should provide visibility into:

- Records processed
- Records rejected
- Quarantine rate
- Pipeline status
- Processing duration
- Source file
- Data-quality results

---

## NFR-06 – Auditability

The platform should make it possible to determine:

- Which source produced a record
- Which source file was processed
- When the data was ingested
- Which schema version was used
- Why a record was rejected

---

## NFR-07 – Security

Secrets, credentials, connection strings, and other sensitive
configuration must not be committed to GitHub.

---

# 5. Data Processing Requirements

## Source Period

The initial implementation should use approximately 12 months of
data from each bike-share operator.

The selected period should include at least one source schema change
where applicable so that schema evolution can be demonstrated.

---

## Processing Layers

The platform must maintain three hard boundaries:

```text
Bronze
  ↓
Silver
  ↓
Gold
