# Big-data-project-using-Spark
This is a project built using Hadoop and this has been optimized to run on very large files.
# CSC8101 Engineering for AI — Spark Coursework

**Student:** Karthick Keeranagere Krishnaiah  
**Student ID:** 250180659  
**Module:** CSC8101 Engineering for AI (2025)  
**Platform:** Google Colab + PySpark

---

## Overview

This project implements a data processing pipeline using **Apache Spark (PySpark)** on the **NYC Taxi Trips dataset**. The pipeline filters, enriches, aggregates, and ranks taxi zone data across multiple dataset sizes, with execution time benchmarking.

---

## Datasets

| Dataset | Description |
|---|---|
| `NYC Taxi Trips` | Recorded taxi trips with distance, passengers, origin/destination zone, and fare |
| `NYC Zones` | Zone lookup table mapping location IDs to zone names |

The trips dataset is available in five sizes: `S`, `M`, `L`, `XL`, `XXL`, stored as Parquet files in Google Drive under `CSC8101_Data/`.

---

## Setup

### Prerequisites

- Google account with Google Drive access
- Google Colab (recommended) or a local PySpark environment

### Installation

The script automatically installs required packages on first run:

```bash
pip install findspark pyspark
```

### Data Setup

Mount Google Drive in Colab and ensure the following directory structure exists:

```
MyDrive/
└── CSC8101_Data/
    └── CSC8101_Data/
        ├── tripdata_2021_07.parquet   # S
        ├── tripdata_2021.parquet      # M
        ├── tripdata_2020_21.parquet   # L
        ├── tripdata_1_6_2019.parquet  # XL / XXL
        └── tripdata_7_12_2019.parquet # XL / XXL
```

---

## Pipeline Structure

### Task 0 — Read Data
Loads the zones CSV from GitHub and the trips Parquet file(s) from Google Drive based on the selected size (`S` to `XXL`).

### Task 1 — Filter Rows

- **Task 1.1** — Removes invalid trips where `trip_distance == 0` or (`passenger_count == 0` and `total_amount == 0`).
- **Task 1.2** — Removes outliers from `trip_distance` and `total_amount` using the **modified z-score** method (threshold T = 3.5, median estimated via `percentile_approx`).

### Task 2 — Compute New Columns

- **Task 2.1** — Joins the trips DataFrame with the zones lookup table to resolve `PULocationID` and `DOLocationID` into human-readable zone names.
- **Task 2.2** — Computes `unit_profitability = total_amount / trip_distance` for each trip.

### Task 3 — Zone Summarisation and Ranking

- **Task 3.1** — Builds a zone-to-zone graph with aggregated `average_unit_profit`, `trips_count`, and `total_passengers` per origin–destination pair.
- **Task 3.2** — Aggregates per origin zone and ranks the **top 10 zones** by:
  - Total trip volume
  - Average unit profitability
  - Total passenger volume

### Task 4 — Execution Time Benchmarking

Runs the full pipeline across all dataset sizes (`S` → `XXL`), both with and without Task 1.2 (outlier removal), recording wall-clock execution times.

---

## Running the Pipeline

To run for a single dataset size, set the `SIZE` variable near the top of the script:

```python
SIZE = 'S'   # Options: 'S', 'M', 'L', 'XL', 'XXL'
trips = init_trips(SIZE)
```

To run the full benchmark across all sizes:

```python
SIZE = ['S', 'M', 'L', 'XL', 'XXL']
# The benchmarking loop at the bottom of the script handles this automatically
```

---

## Performance Analysis

Execution time increases consistently with dataset size. From `S` (~2.9M rows) to `XXL` (~132.4M rows):

- **Without Task 1.2:** ~9.1s → ~160.6s (~18× increase for ~45× more data), demonstrating good Spark scalability.
- **With Task 1.2:** overhead more than doubles at `XXL` (~363.7s), because the modified z-score requires multiple full dataset scans for `percentile_approx` computation.

The outlier removal step (Task 1.2) has the largest per-task performance impact at scale, due to its distributed aggregation requirements.

---

## Project Structure

```
csc8101_spark_coursework_colab.py   # Main script (exported from Colab notebook)
README.md                           # This file
```

---

## Dependencies

| Package | Purpose |
|---|---|
| `pyspark` | Distributed data processing |
| `findspark` | Locates PySpark installation |
| `pandas` | Benchmark results table display |

---

## Notes

- The Spark session is configured with `spark.driver.memory = 10g` to handle larger dataset sizes in Colab.
- `percentile_approx` is used as a scalable surrogate for exact median computation.
- The `pipeline()` function must not be modified — it orchestrates all task functions in order.
