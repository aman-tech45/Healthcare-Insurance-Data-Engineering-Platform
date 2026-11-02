# Hospital Patient Flow Analytics

An end-to-end **real-time hospital patient flow analytics** solution that ingests patient admission and discharge events, processes them through a Medallion Architecture, and exposes curated data for SQL analytics and Power BI reporting.

The project demonstrates how streaming data engineering and analytical reporting can work together to help hospital operations teams monitor patient inflow, bed utilization, length of stay, department activity, and patient demographics.

---

## 1. Business Problem

Hospitals generate continuous operational data as patients are admitted, transferred, and discharged. When this information is spread across systems or processed only in periodic batches, operational teams may not have a timely view of patient flow and bed utilization.

This project builds a data pipeline to:

- Ingest patient-flow events from a streaming source.
- Store raw events in a data lake for traceability.
- Parse, validate, and standardize patient records.
- Build curated dimensional and fact datasets.
- Make analytical data available through SQL views.
- Present operational KPIs through a Power BI dashboard.

> **Note:** This is a portfolio/demo implementation. It uses sample or generated patient-flow data and is not intended for clinical decision-making or production use with real patient information.

## 2. Solution Architecture

```text
Patient Event Producer
        |
        v
Azure Event Hubs
(Kafka-compatible endpoint)
        |
        v
Azure Databricks - Structured Streaming
        |
        v
Azure Data Lake Storage Gen2
        |
        v
+-------------------------------+
|       Medallion Architecture  |
|                               |
|  Bronze - Raw event data      |
|           |                   |
|           v                   |
|  Silver - Parsed and cleaned  |
|           |                   |
|           v                   |
|  Gold - Dimensions and facts  |
+-------------------------------+
        |
        v
Azure Synapse Analytics
(Dedicated SQL pool / external tables)
        |
        v
SQL Views and Business KPIs
        |
        v
Power BI Dashboard
```

### Data flow

1. **Event ingestion:** Patient-flow events are published to Azure Event Hubs. Databricks reads the Event Hubs Kafka-compatible endpoint using Structured Streaming.
2. **Bronze layer:** Incoming Kafka message values are cast to strings and persisted as raw JSON records in Delta format on ADLS Gen2.
3. **Silver layer:** Raw JSON is parsed using an explicit schema. Timestamp fields are converted, selected invalid values are handled, and the cleaned records are written to a Silver Delta location.
4. **Gold layer:** The Silver dataset is transformed into patient and department dimensions and a patient-flow fact table. Patient attribute changes are tracked using a Type 2-style history pattern.
5. **SQL analytics:** External tables expose the Gold datasets through Azure Synapse SQL. SQL views calculate operational KPIs and trends.
6. **Visualization:** Power BI connects to the analytical layer to present patient-flow insights.

## 3. Technology Stack

| Area | Technology |
|---|---|
| Streaming ingestion | Azure Event Hubs |
| Streaming protocol | Kafka-compatible interface |
| Data processing | Azure Databricks, PySpark, Structured Streaming |
| Data lake | Azure Data Lake Storage Gen2 |
| Storage format | Delta Lake |
| Data architecture | Medallion Architecture |
| Data modeling | Fact and dimension tables, SCD Type 2-style history |
| SQL analytics | Azure Synapse Analytics SQL pool |
| Visualization | Power BI |
| Programming / query languages | Python, PySpark, SQL |

## 4. Repository Structure

```text
hospital-patient-flow-analytics/
│
├── databricks-notebooks/
│   ├── 01_bronze_rawdata.py
│   ├── 02_silver_cleandata.py
│   └── 03_gold_transform.py
│
├── sqlpool-quries/
│   ├── SQL_pool_quries.sql
│   └── SQL_views_DDL.sql
│
├── powerbi/
│   └── Hospital_Dashboard.pbix
│
├── .gitignore
└── README.md
```

## 5. Medallion Architecture Implementation

### Bronze Layer - Raw Ingestion

**Notebook:** `databricks-notebooks/01_bronze_rawdata.py`

Responsibilities:

- Connect to the Event Hubs Kafka-compatible endpoint.
- Read streaming messages using Spark Structured Streaming.
- Cast the Kafka `value` field to a string containing the raw JSON event.
- Persist the incoming records in Delta format in ADLS Gen2.
- Maintain a streaming checkpoint location to support recovery.

The Bronze layer preserves the incoming event payload before applying business transformations.

### Silver Layer - Data Cleaning and Standardization

**Notebook:** `databricks-notebooks/02_silver_cleandata.py`

Responsibilities:

- Read the Bronze Delta stream.
- Parse JSON using an explicit PySpark schema.
- Extract patient, department, hospital, bed, admission, and discharge attributes.
- Convert admission and discharge fields into timestamp data types.
- Apply basic handling for invalid or future admission timestamps.
- Demonstrate schema evolution by ensuring expected columns exist.
- Write standardized records to the Silver Delta layer.

### Gold Layer - Dimensional Modeling and Analytics

**Notebook:** `databricks-notebooks/03_gold_transform.py`

Responsibilities:

- Identify the latest patient record using a window function.
- Build a patient dimension containing patient attributes.
- Detect changes in patient attributes using a SHA-256 hash.
- Maintain historical patient records using `effective_from`, `effective_to`, and `is_current` fields.
- Build a department dimension using department and hospital identifiers.
- Create a patient-flow fact table with dimension surrogate keys.
- Derive admission date, length of stay, current-admission indicator, and ingestion timestamp.
- Persist curated Delta datasets for downstream analytics.

**Gold model overview**

**Dimension tables**

| Table | Purpose |
|---|---|
| `dim_patient` | Patient attributes and historical versions |
| `dim_department` | Department and hospital reference data |

**Fact table**

| Table | Purpose |
|---|---|
| `fact_patient_flow` | Patient admission/discharge events and derived operational metrics |

The patient dimension implements a Type 2-style history pattern. The department dimension is built from distinct department and hospital combinations.

## 6. SQL Analytics

The SQL scripts are stored in `sqlpool-quries/`.

### `SQL_pool_quries.sql`

This script demonstrates:

- Creation of a database master key.
- Creation of a database-scoped credential using Managed Identity.
- Configuration of an external data source pointing to ADLS Gen2.
- Definition of an external Parquet file format.
- Creation of external tables for patient, department, and fact datasets.

### `SQL_views_DDL.sql`

The views provide analytical metrics such as:

| View | Business purpose |
|---|---|
| `vw_bed_occupancy` | Bed occupancy percentage, grouped by gender |
| `vw_bed_turnover_rate` | Fact-event count relative to distinct beds |
| `vw_patient_demographics` | Currently admitted patient count by gender |
| `vw_avg_treatment_duration` | Average length of stay by department and gender |
| `vw_patient_volume_trend` | Patient admission volume over time |
| `vw_department_inflow` | Currently admitted patient count by department and gender |
| `vw_overstay_patients` | Patients with a stay exceeding the configured 50-hour threshold |

These are the metrics currently represented by the SQL view definitions. Validate the business definitions against hospital reporting requirements before using them operationally.

## 7. Power BI Dashboard

**File:** `powerbi/Hospital_Dashboard.pbix`

The Power BI report is the visualization layer of the solution. It can be used to explore the curated patient-flow data and communicate operational trends to stakeholders.

Suggested areas to explore in the report:

- Patient admission volume and trends.
- Bed occupancy and utilization.
- Department-wise patient flow.
- Average length of stay.
- Patient demographic breakdown.
- Long-stay / overstay analysis.

Open the `.pbix` file using Power BI Desktop. Configure or refresh its data source according to your Synapse environment and credentials.

## 8. Setup and Execution

### Prerequisites

You will need:

- An Azure subscription.
- An Azure Event Hubs namespace and event hub.
- An Azure Data Lake Storage Gen2 account with a container.
- An Azure Databricks workspace with access to the required Azure resources.
- A compatible Databricks compute environment with the required Kafka and Delta capabilities.
- Azure Synapse Analytics with the required SQL pool and storage access.
- Power BI Desktop.

### Step 1 - Configure Azure resources

Create the Event Hubs namespace and event hub, ADLS Gen2 storage account and container, Databricks workspace, and Synapse SQL pool.

Ensure the relevant identities have the necessary permissions to read from Event Hubs and access the storage account.

### Step 2 - Configure the Databricks notebooks

Import the three Python notebooks from `databricks-notebooks/` into your Databricks workspace.

Replace the configuration placeholders in the notebooks:

- Event Hubs namespace hostname.
- Event hub name.
- Event Hubs connection string.
- ADLS Gen2 storage account name.
- Container and directory paths.
- Storage authentication configuration.

Use Databricks secrets, Unity Catalog storage credentials, or another supported secure authentication method rather than committing credentials or access keys to source control.

**Important:** The notebooks currently contain placeholder values. They will not run until the resource names, paths, and authentication are configured for your environment.

### Step 3 - Run the pipeline

Run the notebooks in the following order:

1. `01_bronze_rawdata.py` - start the streaming ingestion into Bronze.
2. `02_silver_cleandata.py` - start the Bronze-to-Silver streaming transformation.
3. `03_gold_transform.py` - create/update the Gold dimensional and fact datasets.

The Bronze and Silver notebooks use streaming writes. The Gold notebook currently uses batch reads and writes, so it should be executed after Silver has received data. For a continuously refreshed Gold layer, schedule or redesign this transformation as an incremental job.

### Step 4 - Configure Synapse SQL

1. Open `sqlpool-quries/SQL_pool_quries.sql`.
2. Replace the storage account, container, and password placeholders.
3. Configure the database-scoped credential and external data source for your environment.
4. Create the external tables after the Gold datasets are available in the expected storage locations.
5. Verify that the external table column definitions, file formats, and storage paths match the actual Gold output.

### Step 5 - Create SQL views

Execute `sqlpool-quries/SQL_views_DDL.sql` after the external tables have been created.

Validate each view's output against sample records and the intended business definition.

### Step 6 - Open the Power BI report

1. Open `powerbi/Hospital_Dashboard.pbix` in Power BI Desktop.
2. Configure the Synapse connection and authentication if required.
3. Refresh the model.
4. Validate the report metrics against the SQL views.

## 9. Data Model - Key Attributes

The pipeline uses the following patient-flow attributes:

| Attribute | Description |
|---|---|
| `patient_id` | Natural identifier for a patient |
| `gender` | Patient demographic attribute |
| `age` | Patient age |
| `department` | Hospital department |
| `hospital_id` | Hospital identifier |
| `bed_id` | Bed identifier |
| `admission_time` | Patient admission timestamp |
| `discharge_time` | Patient discharge timestamp |
| `length_of_stay_hours` | Derived duration between discharge and admission |
| `is_currently_admitted` | Derived current-admission indicator |

The Gold fact table also stores patient and department surrogate keys to support dimensional analysis.

## 10. Engineering Considerations and Future Improvements

The current repository is a working demonstration of the architecture and contains areas that should be strengthened before production deployment:

- **Secure authentication:** Replace account-key configuration with managed identity, Unity Catalog storage credentials, or another supported secret-based approach. Never commit connection strings, keys, or passwords.
- **Data quality:** Add explicit checks for null identifiers, invalid ages, missing timestamps, negative length of stay, and invalid department or bed references. The current Silver notebook replaces ages above 100 with random values; this should be replaced with a documented quarantine or correction rule.
- **Event-time correctness:** Define how late-arriving and out-of-order events are handled. Validate admission and discharge timestamps rather than automatically assigning the current timestamp to invalid admissions.
- **SCD Type 2 reliability:** Use deterministic surrogate keys and an idempotent merge strategy. Review the current Gold implementation for duplicate processing and rerun safety.
- **Department dimension:** Add an incremental upsert strategy instead of overwriting the dimension on each execution.
- **Fact-table processing:** Implement incremental processing and deduplication so reruns do not duplicate fact records. Review the current fact write mode, which overwrites the target.
- **Streaming orchestration:** Add job dependencies, monitoring, retry policies, and alerts for failed or delayed streams.
- **Performance:** Review partitioning, file sizes, compaction, and query performance as data volumes grow.
- **Metric definitions:** Confirm whether occupancy, turnover, active admissions, and overstay metrics match the operational definitions used by the hospital.
- **Privacy:** Use synthetic or de-identified data for demonstrations and apply appropriate access controls to any sensitive data.

## 11. Learning Outcomes

This project demonstrates practical exposure to:

- Real-time ingestion using Azure Event Hubs and Kafka-compatible streaming.
- PySpark Structured Streaming in Azure Databricks.
- Delta Lake storage and Medallion Architecture.
- Data parsing, cleaning, and schema evolution.
- Dimensional modeling and Type 2-style historical tracking.
- SQL-based analytical modeling in Azure Synapse.
- Building a BI reporting layer with Power BI.
- Thinking through data quality, reliability, security, and pipeline operations.

## 12. Disclaimer

This repository is intended for educational and portfolio demonstration purposes. It is not a certified hospital information system and should not be used to make clinical, patient-care, staffing, or emergency decisions.

---

**Author:** Aman Jha

**Project:** Hospital Patient Flow Analytics
