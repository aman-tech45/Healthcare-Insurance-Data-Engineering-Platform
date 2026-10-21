# Healthcare Insurance Data Engineering Platform

An end-to-end cloud data engineering project designed to integrate healthcare insurance data from multiple source systems, improve data quality, and deliver analytics-ready datasets using Microsoft Azure.

## Overview

Healthcare and insurance data is often distributed across multiple systems and formats, making it difficult to maintain consistent and reliable datasets for reporting.

This project demonstrates a data integration and transformation architecture using Azure Data Factory, Azure Data Lake Storage Gen2, Azure Databricks, PySpark, and Delta Lake. It follows the Medallion Architecture to progressively transform raw data into clean, standardized, and business-ready datasets.

## Architecture

```text
MySQL Hospital Data ───────┐
MongoDB / Cosmos DB Claims ├──> Azure Data Factory
Azure Blob CSV Files ──────┘            |
                                        v
                                  ADLS Gen2
                                  Bronze Layer
                                        |
                                        v
                               Azure Databricks
                                  PySpark
                                        |
                                        v
                                  Silver Layer
                                        |
                                        v
                                Gold Transformations
                                        |
                              ┌─────────┴─────────┐
                              v                   v
                       Azure SQL DB            Power BI
```

## Technology Stack

| Technology | Purpose |
|---|---|
| Azure Data Factory | Data ingestion and pipeline orchestration |
| Azure Data Lake Storage Gen2 | Central storage for Bronze, Silver, and Gold data |
| Azure Databricks | Distributed data processing |
| PySpark | Data cleansing, validation, and transformation |
| Delta Lake | Reliable lakehouse table storage |
| Azure Key Vault | Secure management of credentials and secrets |
| Self-Hosted Integration Runtime | Connectivity to on-premises MySQL |
| Azure SQL Database | Storage for selected analytical datasets |
| Azure Blob Storage | Source CSV files |
| Power BI | Reporting and visualization |

## Source Systems

The project integrates the following healthcare insurance datasets:

- **On-premises MySQL:** Hospital data.
- **MongoDB / Cosmos DB:** Claims data.
- **Azure Blob Storage:** Disease, Patient, Group, Subgroup, and Subscriber CSV files.

## Medallion Architecture

### 1. Bronze Layer — Raw Data

The Bronze layer stores ingested source data in ADLS Gen2 with minimal transformation.

Key activities:
- Ingest data through Azure Data Factory Copy activities.
- Preserve original source information for traceability and reprocessing.
- Organize data by source and dataset.
- Maintain a reliable input layer for downstream processing.

### 2. Silver Layer — Cleansed Data

Azure Databricks and PySpark process Bronze data to improve quality and consistency.

Key transformations:
- Remove duplicate records.
- Handle null and invalid values.
- Standardize column names, values, and data formats.
- Validate records against applicable data-quality rules.
- Normalize multi-valued fields using `split()` and `explode()`.
- Write the processed datasets to the Silver layer.

### 3. Gold Layer — Analytics-Ready Data

The Gold layer contains curated datasets designed for reporting and business analysis.

Key activities:
- Join related Silver datasets using appropriate business keys.
- Select relevant business attributes.
- Create derived columns and aggregated metrics.
- Apply business-specific transformation rules.
- Publish selected outputs for Azure SQL Database and Power BI consumption.

## Azure Data Factory Pipelines

The ingestion architecture uses three principal pipelines:

| Pipeline | Source | Destination |
|---|---|---|
| `OnPrem2Bronze_PL` | On-premises MySQL | ADLS Gen2 Bronze |
| `Mongo2Bronze_PL` | MongoDB / Cosmos DB | ADLS Gen2 Bronze |
| `Raw2Bronze_PL` | Azure Blob Storage CSV files | ADLS Gen2 Bronze |

The CSV ingestion pipeline uses the wildcard pattern `*.csv` to process multiple files from the configured source folder.

ADF linked services define source and destination connectivity. A Self-Hosted Integration Runtime supports connectivity between ADF and the on-premises MySQL environment.

## Databricks Processing and Reusable Code

Transformation logic is organized into common notebooks and dataset-specific notebooks.

Common notebooks:
- Connector functions.
- Generic transformation functions.
- Reusable data-processing logic.

Dataset-specific notebooks process datasets such as Claims, Hospital, Disease, Patient, Group, Subgroup, and Subscriber.

Common notebooks can be reused with Databricks `%run`:

```python
%run "/Path/Connector"
%run "/Path/Generic_Functions"
```

This design reduces duplicated code and makes transformation logic easier to maintain.

## Orchestration

Azure Data Factory orchestrates the overall ingestion and transformation workflow.

The end-to-end execution follows this sequence:

1. Ingest data from configured source systems.
2. Store the ingested data in the Bronze layer.
3. Invoke Databricks notebooks after successful ingestion.
4. Process and validate the required Silver datasets.
5. Execute dependent Gold transformations.
6. Publish the required analytical outputs.

Notebook dependencies must be configured explicitly. Independent tasks may run in parallel when resource capacity and output paths permit. Tasks that write to shared destinations should be coordinated to prevent concurrent-write conflicts.

## Security and Reliability

The architecture incorporates the following practices:

- Azure Key Vault for storing sensitive connection information.
- Managed identity where supported by the configured Azure services.
- Restricted permissions for storage, secrets, and databases.
- Data validation before downstream consumption.
- Pipeline execution monitoring and failure investigation.
- Preservation of Bronze data to support reprocessing.

## Repository Structure

```text
healthcare-insurance-data-engineering/
├── README.md
├── data/
│   └── sample/                  # Synthetic sample data only
├── adf/
│   ├── pipelines/
│   ├── datasets/
│   └── linked-services/
├── databricks/
│   ├── common/
│   │   ├── Connector
│   │   └── Generic_Functions
│   ├── development/
│   └── production/
├── pyspark/
│   ├── bronze_to_silver/
│   └── silver_to_gold/
├── sql/
│   └── gold/
├── tests/
│   └── data_quality/
└── docs/
    └── architecture.md
```

This is a suggested repository structure; adapt it to match the actual exported Azure resources and notebook files.

## Setup and Execution

1. Configure the required Azure resources and permissions.
2. Create an ADLS Gen2 storage account with the required containers or directory structure.
3. Configure ADF linked services, datasets, and the appropriate Integration Runtime.
4. Configure source access and ingest the source datasets into Bronze.
5. Create an Azure Databricks workspace and compute environment.
6. Configure secure access to the required Azure resources.
7. Import the common and dataset-specific notebooks.
8. Execute and validate Silver transformations.
9. Build the required Gold datasets and publish selected outputs.
10. Configure the end-to-end ADF pipeline and verify its execution.

The actual connection strings, storage paths, credentials, schemas, and source configurations must be supplied for the target environment.

## Data Quality and Validation

Suggested validation checks include:

- Null checks on required fields.
- Duplicate checks using appropriate business keys.
- Record-count reconciliation between source and target.
- Data-type and format validation.
- Referential integrity checks where relationships are defined.
- Validation of business rules before publishing Gold outputs.

## Expected Outcomes

- Centralized ingestion from multiple source systems.
- Reusable PySpark transformation logic.
- Structured Bronze, Silver, and Gold data layers.
- Improved data consistency and traceability.
- Automated orchestration of dependent processing steps.
- Analytics-ready datasets for downstream SQL and BI consumption.
