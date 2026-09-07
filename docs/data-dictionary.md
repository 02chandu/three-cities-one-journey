# Data Dictionary

## Silver Layer

### 1. silver_trips
| Column                  | Data Type | Purpose                                    |
| ----------------------- | --------- | ------------------------------------------ |
| `trip_id`               | STRING    | Unique identifier for a trip               |
| `source`                | STRING    | Source operator: `citibike` or `divvy`     |
| `city`                  | STRING    | City where the trip occurred               |
| `started_at`            | TIMESTAMP | Standardized trip start timestamp          |
| `ended_at`              | TIMESTAMP | Standardized trip end timestamp            |
| `start_station_id`      | STRING    | Standardized starting station ID           |
| `start_station_name`    | STRING    | Starting station name                      |
| `end_station_id`        | STRING    | Standardized ending station ID             |
| `end_station_name`      | STRING    | Ending station name                        |
| `start_lat`             | DOUBLE    | Starting station latitude                  |
| `start_lon`             | DOUBLE    | Starting station longitude                 |
| `end_lat`               | DOUBLE    | Ending station latitude                    |
| `end_lon`               | DOUBLE    | Ending station longitude                   |
| `trip_duration_seconds` | BIGINT    | Trip duration in seconds                   |
| `bike_type`             | STRING    | Bike type where available                  |
| `member_type`           | STRING    | User/member classification where available |
| `weather_hour`          | TIMESTAMP | Hour used for weather matching             |
| `ingestion_date`        | DATE      | Date the record entered the platform       |
| `source_file`           | STRING    | Original source file for lineage           |


### 2. silver_stations
| Column           | Data Type | Purpose                                      |
| ---------------- | --------- | -------------------------------------------- |
| `station_key`    | STRING    | Conformed internal station identifier        |
| `source`         | STRING    | Original operator                            |
| `city`           | STRING    | City                                         |
| `station_id`     | STRING    | Source station ID                            |
| `station_name`   | STRING    | Station name                                 |
| `latitude`       | DOUBLE    | Station latitude                             |
| `longitude`      | DOUBLE    | Station longitude                            |
| `valid_from`     | DATE      | Start date for this station identity/version |
| `valid_to`       | DATE      | End date for this station identity/version   |
| `is_current`     | BOOLEAN   | Indicates current station record             |
| `source_file`    | STRING    | Source-file lineage                          |
| `ingestion_date` | DATE      | Ingestion date                               |


### 3. silver_weather
| Column           | Data Type | Purpose                       |
| ---------------- | --------- | ----------------------------- |
| `city`           | STRING    | City                          |
| `weather_hour`   | TIMESTAMP | Standardized hourly timestamp |
| `temperature`    | DOUBLE    | Temperature                   |
| `precipitation`  | DOUBLE    | Precipitation                 |
| `wind_speed`     | DOUBLE    | Wind speed                    |
| `weather_code`   | INT       | Weather condition/code        |
| `source`         | STRING    | Weather source                |
| `ingestion_date` | DATE      | Ingestion date                |
| `source_file`    | STRING    | Lineage/source information    |

## Data Modeling Principles

- Silver provides a conformed model across source systems.
- Source-specific differences are resolved during Silver processing.
- Source station identifiers are retained for lineage.
- A conformed station key is used for analytical consistency.
- Weather is aligned to the trip start hour.
- Source lineage is retained through source and source-file attributes.
- Fields that are unavailable from a source are not artificially populated.
