# Healthcare Insurance Data Engineering Platform

An end-to-end data engineering project to integrate healthcare insurance data from multiple source systems, improve data quality, and prepare analytics-ready datasets using Microsoft Azure.

> **Project status:** Repository scaffold. Implementation code and Azure resource exports will be added as the project is built. Do not describe unimplemented components as completed.

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
                               Azure Databricks / PySpark
                                        |
                                        v
                                  Silver Layer
                                        |
                                        v
                                  Gold Layer
                                        |
                              ┌─────────┴─────────┐
                              v                   v
                       Azure SQL DB            Power BI
```

## Technology Stack

- **Ingestion and orchestration:** Azure Data Factory
- **Cloud storage:** Azure Data Lake Storage Gen2, Azure Blob Storage
- **Processing:** Azure Databricks, PySpark
- **Lakehouse storage:** Delta Lake
- **Secrets:** Azure Key Vault
- **Connectivity:** Self-Hosted Integration Runtime for on-premises MySQL, where required
- **Analytics:** Azure SQL Database, Power BI

## Source Datasets

- MySQL: hospital data
- MongoDB / Cosmos DB: claims data
- CSV files: disease, patient, group, subgroup, and subscriber data

Use synthetic or appropriately de-identified sample data in this public repository. Never commit real patient or insurance records.

## Medallion Layers

### Bronze — Raw data
- Ingest source data with minimal transformation.
- Preserve source information for traceability and reprocessing.
- Organize files by source and dataset.

### Silver — Cleansed data
- Handle null and invalid values.
- Standardize columns and data formats.
- Deduplicate using appropriate business keys.
- Validate records against documented data-quality rules.
- Normalize comma-separated identifiers when required.

### Gold — Analytics-ready data
- Join related Silver datasets using verified business keys.
- Create derived attributes and business aggregations.
- Publish selected datasets for SQL and BI consumption.

## Planned ADF Pipelines

| Pipeline | Source | Destination |
|---|---|---|
| `OnPrem2Bronze_PL` | On-premises MySQL | ADLS Gen2 Bronze |
| `Mongo2Bronze_PL` | MongoDB / Cosmos DB | ADLS Gen2 Bronze |
| `Raw2Bronze_PL` | Azure Blob CSV files | ADLS Gen2 Bronze |

The actual connector configuration depends on the source technology, network access, and Azure services available in the implementation environment.

## Repository Structure

```text
healthcare-insurance-data-engineering/
├── README.md
├── data/
│   └── sample/
├── adf/
│   ├── pipelines/
│   ├── datasets/
│   └── linked-services/
├── databricks/
│   ├── common/
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

Empty folders contain `.gitkeep` files until implementation files are added.

## Planned Implementation Steps

1. Define source schemas and create synthetic sample datasets.
2. Configure ADLS Gen2 storage and access permissions.
3. Build ADF ingestion pipelines and validate Bronze outputs.
4. Develop and test reusable PySpark transformations.
5. Write validated datasets to Silver and create Gold models.
6. Configure orchestration, dependencies, monitoring, and failure handling.
7. Publish selected outputs to Azure SQL and build Power BI reporting.
8. Document execution steps, test results, and architecture decisions.

## Data Quality Checks

Planned checks include:
- Required-field and null checks
- Duplicate detection using business keys
- Source-to-target record-count reconciliation
- Data type and format validation
- Referential integrity where relationships are defined
- Business-rule validation

## Security

- Store secrets in Azure Key Vault or an approved secrets manager.
- Use managed identity where supported and configured.
- Apply least-privilege permissions.
- Do not commit credentials, access keys, connection strings, or sensitive data.

## Future Enhancements

- Metadata-driven ingestion
- Incremental loading and CDC where supported
- Automated data-quality reporting and alerts
- CI/CD for Azure Data Factory and Databricks assets
- Pipeline observability and operational metrics
- Sensitive-data masking and role-based access controls

## Project Status

This repository currently provides the project structure and implementation plan. Update this section as each component is implemented and tested.
