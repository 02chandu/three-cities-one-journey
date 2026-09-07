# Design Decisions

## 1. Why Medallion Architecture?

The platform uses three clearly separated layers:

Each layer has a specific responsibility.

Bronze

Raw source data.

Silver

Conformed and validated data.

Gold

Business-ready analytical data.

This separation makes the pipeline easier to understand, test, troubleshoot,
and maintain.

## 2. Why Keep Bronze Immutable?

Bronze represents the original source data.

Therefore, Bronze should not be transformed or overwritten.

Benefits:

Original data can be audited.
Data can be reprocessed.
Transformation logic can change without losing source data.
Source-to-target lineage is easier to establish.
Failed downstream processing can be replayed.

Decision:

Never modify previously ingested Bronze data.

## 3. Why Use Azure Data Factory?

Azure Data Factory is responsible for:

Source ingestion
Pipeline orchestration
Scheduling
Passing configuration
Triggering downstream processing
Monitoring pipeline execution

Transformation logic is intentionally kept outside ADF.

Decision:

ADF handles orchestration and ingestion while Databricks handles
transformation.

## 4. Why Use ADLS Gen2?

ADLS Gen2 provides centralized cloud storage for the data platform.

It will contain:

Bronze
Silver
Gold
Quarantine

This provides a clear physical separation between processing stages.

Decision:

ADLS Gen2 is the central storage layer for the platform.

## 5. Why Use Azure Databricks?

The project requires transformation across multiple source systems
with different schemas and data definitions.

Databricks provides the processing environment for PySpark transformations.

Responsibilities include:

Schema handling
Data standardization
Data-quality validation
Conformance
Weather enrichment
Gold-mart creation

Decision:

Databricks is responsible for transformation and analytical data
processing.

## 6. Why Use PySpark?

The project is designed as a Data Engineering platform rather than
a simple single-file Python script.

PySpark provides a distributed processing model that can scale as the
dataset grows.

Decision:

PySpark is used as the primary transformation language inside
Databricks.

## 7. Why Use a Schema Registry?

The source systems can change their schemas over time.

Without a schema registry, source-specific changes can lead to
hardcoded logic throughout the pipeline.

The schema registry provides:

Schema version
Source
Period
Expected columns
Expected data types
Required fields

Decision:

Source schema changes should be represented through configuration
rather than unnecessary changes to core transformation logic.

## 8. Why Quarantine Instead of Dropping Invalid Records?

Invalid records should not disappear silently.

Instead, they are moved to a quarantine area together with information
about why they failed validation.

Example:

Invalid Record
      ↓
Quarantine
      ↓
Reason + Source + File + Processing Time

Benefits:

Easier debugging
Better auditability
Better data-quality monitoring
Ability to investigate source problems

Decision:

Invalid records are quarantined rather than silently dropped.

## 9. Why Use Quality Gates?

A data-quality check is useful only if its result affects pipeline behavior.

Critical quality failures should prevent invalid data from being
published to the next layer.

Example:

Bronze
  ↓
Quality Gate
  ↓
PASS → Silver
FAIL → Stop / Quarantine

Decision:

Quality gates protect downstream layers from invalid data.

## 10. Why Normalize Timestamps?

Citi Bike, Divvy, and weather data may use different timestamp
representations.

To correctly join trip data with weather data, timestamps must be
standardized.

The platform will account for:

Timestamp formats
Timezones
Daylight Saving Time
Hour-level weather alignment

Decision:

Timestamp normalization occurs during Silver processing.

## 11. Why Weather Enrichment in Silver?

Weather is an enrichment of the trip data rather than a final
visualization-only calculation.

Joining weather during the curated-data stage allows multiple Gold
marts to reuse the enriched data.

Decision:

Weather enrichment is performed as part of the Silver processing
flow.

## 12. Why Three Gold Marts?

The project requires multiple analytical perspectives.

The initial marts are:

Station-Hour Demand
Normalized Daily City Comparison
Weather Sensitivity

This separates analytical use cases while keeping the Silver layer
focused on reusable conformed data.

Decision:

Gold contains business-oriented datasets derived from the common
Silver model.

## 13. Why Idempotency?

A pipeline may be retried because of:

Temporary failures
Network issues
Infrastructure failures
Manual reruns

Repeated execution should not create duplicate records.

Decision:

Pipeline operations must be designed to be safely rerunnable.

## 14. Why Configuration Over Hardcoding?

Source-specific behavior should be controlled through configuration
where practical.

Examples:

Source name
Schema version
File pattern
Column mappings
Required fields

Benefits:

Easier maintenance
Easier schema evolution
Less duplicated code
Easier onboarding of additional sources

Decision:

Configuration should control source-specific behavior instead of
spreading hardcoded values throughout the pipeline.

## 15. Why Separate Power BI from Transformation?

Power BI is the consumption layer.

The major transformation and conformance logic belongs upstream.

Therefore:

Source
  ↓
ADF
  ↓
ADLS
  ↓
Databricks
  ↓
Gold
  ↓
Power BI

Decision:

Power BI consumes curated Gold data instead of becoming the primary
transformation engine.

## 16. Why GitHub?

GitHub is used for:

Version control
Source code management
Configuration management
Documentation
Change history
Project demonstration

The repository should demonstrate the engineering process rather than
only the final code.

## 17. Security Decision

Secrets must never be stored in GitHub.

Examples:

Passwords
Access keys
Connection strings
API secrets
Tokens

Production credentials will be handled through appropriate Azure
security mechanisms rather than hardcoded into notebooks or pipelines.

## 18. Overall Architecture Decision

The final responsibility split is:

Component	Responsibility
Source Systems	Provide raw data
Azure Data Factory	Ingestion and orchestration
ADLS Gen2	Storage
Databricks	Transformation
Schema Registry	Schema configuration/versioning
Quarantine	Invalid records
Silver	Conformed/enriched data
Gold	Analytical datasets
Power BI	Visualization
GitHub	Version control and documentation

The architecture is designed around clear layer boundaries,
reproducibility, data quality, configuration, and maintainability.
