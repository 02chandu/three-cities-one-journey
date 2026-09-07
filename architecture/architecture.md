# Architecture Design

## 1. Architecture Overview

The Three Cities, One Journey platform follows a Medallion architecture
with three clearly separated processing layers:

```text
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
