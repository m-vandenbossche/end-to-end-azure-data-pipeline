# Pipeline Architecture

The data pipeline is orchestrated using **Azure Synapse Pipelines** following a **Bronze → Silver → Gold** architecture. A master pipeline controls the execution flow, ensuring each stage only begins after the previous one has completed successfully.

## Pipeline Flow

```text
Master Pipeline
│
├── Blob → Bronze
│
├── Bronze → Silver
│   ├── Get Metadata
│   ├── ForEach File
│   ├── Archive Raw Files
│   ├── Copy to Silver
│   └── Execute Cleaning Notebook
│       ├── Clean & standardize telemetry
│       ├── Detect anomalies (Isolation Forest)
│       ├── Weather enrichment (BrightSky API)
│       └── Write Silver Parquet
│
└── Silver → Gold
    ├── Load Silver datasets
    ├── Align schemas
    ├── Join on UTC timestamps
    └── Write Gold dataset
```

---

## Blob → Bronze

Raw telemetry CSV files are ingested into the **Bronze** container. This layer serves as the landing zone and preserves the original data without modification, providing a reliable source for auditing and reprocessing.

---

## Bronze → Silver

The **Bronze → Silver** pipeline prepares the telemetry data for analytics.

### Pipeline Activities

- Detect newly ingested telemetry files using **Get Metadata**
- Iterate over each file using a **ForEach** activity
- Archive the original raw files
- Copy files to the Silver processing area
- Execute the `BeeHive-cleaning` notebook to:
  - Clean and standardize sensor measurements
  - Detect anomalous readings using **Isolation Forest** (`scikit-learn`)
  - Enrich telemetry with historical weather data from the **BrightSky Weather API**
  - Store the processed data as **Parquet** files in the Silver layer
- Remove successfully processed files from the Bronze landing zone

---

## Silver → Gold

The `Silver_to_gold` notebook creates the final analytics-ready dataset.

### Processing Steps

- Load all Silver Parquet datasets
- Align schemas across different sensor types
- Merge datasets using UTC timestamps
- Write the consolidated dataset to the Gold container

The resulting Gold dataset provides a single, analytics-ready source for reporting, visualization, and downstream machine learning workloads.

---

## Technologies Used

| Category | Technology |
|----------|------------|
| Cloud Platform | Azure Synapse Analytics |
| Orchestration | Azure Synapse Pipelines / Azure Data Factory |
| Processing | PySpark, Pandas |
| Machine Learning | scikit-learn (Isolation Forest) |
| Storage | Azure Blob Storage, Parquet |
| External Data | BrightSky Weather API |