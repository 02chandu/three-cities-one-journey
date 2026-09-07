# Schema Registry

## Purpose

The schema registry defines the expected structure of source datasets
for each source and processing period.

It allows the pipeline to handle source schema changes through
configuration instead of hardcoding schema-specific logic throughout
the transformation code.

---

## Registry Structure

schema_registry/
│
├── README.md
│
├── citibike/
│   ├── schema_v1.json
│   └── schema_v2.json
│
├── divvy/
│   └── schema_v1.json
│
└── weather/
    └── schema_v1.json

Registry Key

Each schema definition is associated with:

Source
Period
Schema version

Example:

source = citibike
period = 2025-01
schema_version = v1
Schema Definition

Each schema definition should describe:

Column name
Expected data type
Required/optional status
Source-specific field information

Example:

{
  "source": "example",
  "period": "2025-01",
  "schema_version": "v1",
  "columns": [
    {
      "name": "example_column",
      "type": "string",
      "required": true
    }
  ]
}

The actual schemas will be created after we inspect the real source
files.

Schema Evolution

If a source changes its schema:

New Source Schema
        ↓
Identify Schema Version
        ↓
Add Registry Entry
        ↓
Validate Source Data
        ↓
Apply Conformance Logic

The objective is to avoid unnecessary changes to the core pipeline
when a source introduces a new schema version.

Validation

During ingestion/transformation, source data will be checked against
the registered schema.

Potential validation failures include:

Missing required columns
Unexpected data types
Invalid field structure
Unsupported schema version

Critical schema failures should prevent invalid data from being
published downstream.

Design Principle

Schema changes should be managed through the schema registry wherever
possible rather than by spreading source-specific logic throughout
the pipeline.
