Source tag : https://hp.sharepoint.com/:w:/r/teams/DataOps310/Shared%20Documents/Data%20Ops%20documents/Documents/Understanding%20docs/Cumulus_Data_Products_Master_Document.docx?d=wbc240a55a9cc4a8591c6beed498c404e&csf=1&web=1&e=5z1gm4
Note : This extracted file is generated from SharePoint or Wiki content. If the source document contains images, charts, or graphical elements, they may not be fully converted into Markdown format. For complete clarity, please refer to the original source using the source tag provided above.

# Cumulus Data Products Master Document

## Topics Covered
- Cumulus Data Products Overview
- Cumulus ETL Standard Raw
- Cumulus – OCV Process & KLO Monitoring

**Date:**
- 17-Feb-2026
- 20-Feb-2026
- 25-Feb-2026

## Table of Contents
1. Cumulus Data Products Overview
2. Cumulus ETL Standard Raw
3. Cumulus – OCV Process & KLO Monitoring
4. Upstream/ Downstream details
5. Production job schedules/Runbook
6. Point of contact for failures

---

## 1. Cumulus Data Products Overview

### 1.1 Overview of HP DataOS and Data Products
HP DataOS is a centralized data ecosystem that manages and processes large volumes of telemetry and enterprise data.

Within this ecosystem, there are approximately 800 data products. Each data product is designed to serve a specific business purpose. Cumulus is one such data product, primarily focused on printer telemetry data for the Supplies business.

HP printers continuously generate telemetry data such as pages printed, ink drops ejected, toner usage, device status, error logs, and operational metrics.

This data is transmitted to cloud services through firmware and driver mechanisms.

Telemetry data is collected from various sources including home printers, office printers, enterprise printers, smart applications, and related systems.

### 1.2 Architecture
We have 3 major layers present in Data OS:
1. Ingestion
2. ETL (Cumulus)
3. DST

### 1.3 Ingestion layer
There are two types of ingestion in Data OS:

#### 1.3.1 The Legacy Data Model (LEDM)
- Captures printer data in snapshot format at specific points in time, typically using XML-based payloads.
- Legacy data ingestion is done by Pepto.
- Models: LEDM, MHIT, AMPV
- Sources: Pony Express, Gotham, JAM

#### 1.3.2 The Common Data Model (CDM)
- Modern, event-driven model where printer events are captured in real time.
- Uses schema registry as the source of truth and follows standardized event structures.
- Sources: PSDR, Smart app, Smart driver

**Overall Process:**
- Telemetry data first lands in a landing bucket managed by the ingestion team.
- Privacy tagging and consent validation are applied before the data is forwarded for further processing.
- Privacy compliance is handled at multiple layers to ensure regulatory adherence.
- Geo information is added and a payload is created for archival.
- Pepto and Bronze++ have similar processes of data ingestion with differences in schema and tables.

### 1.4 ETL layer (Cumulus)
Cumulus functions as the ETL layer within DataOS. It reads data from the ingestion layer for both CDM and LEDM and performs the following operations:
- Removing duplicates
- Validates product identifiers
- Apply transformation rules
- Extracts required attributes from payloads
- Archival storing

#### 1.4.1 LEDM
- Data from Pepto is processed into STDRAW.
- Structured and stored in Delta Lake tables using Unity Catalog.
- Intermediate raw storage optimized out.

#### 1.4.2 CDM
- Data read from Bronze++ and loaded to PSU (Print Supply Usage).

#### OCV – Online Cartridge Validation
- Separate validation pipeline verifying cartridges used in printers are genuine HP cartridges.
- Communicates with a validation service and stores authenticated results.
- Crucial for business analytics, especially in calculating usage share and HP supply market share.

### 1.5 DST Layer
- Data from STDRAW and PSU combined is landed into DST for analytics processing.
- Within DST: IDST (Ink), TRIDENT (toner/laser).
- ETL layer focuses on data flattening, cleansing; DST layer focuses on aggregations and enrichment.
- Processed DST data is used by BDBT team for analytics and dashboards on Thoughtspot.

### 1.6 Enterprise Data Enrichment
- Printer telemetry data is enriched with enterprise data: shipment details, customer registration, geolocation, product family info, logistics.
- Enables insights: install base, usage trends, forecasting, quarterly market share reporting.

### 1.7 Data Quality and Monitoring
- Data quality checks embedded within ingestion pipelines.
- Automated validation rules: record counts, schema consistency, threshold limits, business logic compliance.
- Operational metrics logged in Unity Catalog tables.
- ML models predict seasonal trends.
- Quality team monitors data quality from ingestion, ETL, DST, and other teams.

### 1.8 Archival Strategy and Data Retention
- Historical data retained for compliance and analytics.
- Older data moved to archival storage for cost reduction while preserving accessibility.
- STDRAW contains data from 2003 with increasing sources.
- CDM retains 3 years of data.
- Dyns and non-dyns: dyns have retention policy (5 years for ink, 3 years for laser); non-dyns are fully retained.

### 1.9 Modernization and Future Enhancements
- Migrating pipelines to serverless environments (Scala to PySpark).
- Standardizing ingestion through unified data models.
- Plan to migrate Pepto to CDM model.
- Automation: data product generation agents, enhanced anomaly detection.
- Focus on improved data quality intelligence and automated monitoring.

---

## 2. Cumulus Data Product – ETL Standard Raw Process

### 2.1 Overview of the Cumulus Standard Raw Process
- Core ETL component within Cumulus data product.
- Handles telemetry data from legacy LEDM sources via Pepto ingestion.
- Processes ~700 GB daily across multiple sources.
- 14 parallel job runs: 4 ink, 10 laser printer sources.
- PSUJ, OCV, XREF are major downstream for STDRAW.
- All jobs complete within three hours.

### 2.2 Data Sources and Input Format
- Source: Pepto landing layer.
- Payload received as XML stored in Unity Catalog tables as string fields.
- Schema varies for ink and laser printers.

### 2.3 Job Architecture and Execution Flow
- ETL workflow: Main Body, BAT, Component BAT.
- Common notebook for ink and one for laser, driven by source ID and metadata.
- Metadata Excel files in Unity Catalog drive column extraction and transformation logic.
- Over 1,200 columns extracted from payload data.
- Data written into Unity Catalog Delta tables.
- Dines (rarely accessed XML columns) and Non-Dines (frequently used structured columns) separated for storage optimization.
- Liquid clustering and Z-standard compression reduced storage from 800 TB to 250 TB.

### 2.3.2 BAT – Data Profiling and Quality Reporting
- BAT (Basic Acceptance Testing) runs after main body.
- Data profiling, compares source and target counts, validates null percentages, checks duplicates.
- Results in JSON and HTML reports sent via email.
- Failures may raise warnings or errors; critical failures may require reprocessing.

### 2.3.3 Component BAT – Business Logic Revalidation
- Samples 25–50% of loaded data, reapplies transformation logic.
- Verifies derived columns, error flags, hash fields, downstream critical attributes.
- Issues mostly from source data inconsistencies.

### 2.4 Failure Handling and Monitoring
- Job failures mostly due to upstream data issues or infrastructure limitations.
- Delta Lake versioning enables rollback and overwrite.
- Monitoring via Databricks workflows, email reports, dashboards.
- KLO agents auto-detect failures and re-trigger pipelines.

### 2.5 Focus on Data Quality and Future Improvements
- Enhance anomaly detection with intelligent agents.
- Build predictive data quality monitoring beyond static validation rules.

---

## 3. Cumulus Data Product – OCV Pipeline and KLO Monitoring

### 3.1 Overview of the OCV Process
- ETL process verifying authenticity of printer cartridges.
- Works with laser printer telemetry data; ink cartridge validation handled separately.
- Consumes processed telemetry data from Standard Raw ETL pipeline.
- Caching, parallel API calls, staged data transformations for scalable processing.

### 3.2 Data Sources and Input Format
- Input: telemetry data from Standard Raw ETL.
- Contains printer-level info, cartridge attributes (serial numbers, product IDs, install dates, manufacturer).
- Cartridges represented by color identifiers (K, C, M, Y).
- Only relevant columns selected for verification.

### 3.3 Pipeline Architecture and Execution Flow
Stages:
1. Snapshot Table Creation
2. Cache Table Comparison
3. OCV Service Verification
4. Export Table Transformation
5. History Table Storage

#### 3.3.1 Snapshot Table Processing
- Temporary staging table during pipeline execution.
- Extracts required fields, reduces dataset size.
- Each row: single printer record with multiple cartridges.
- Only latest records processed; table cleared and recreated each cycle.

#### 3.3.2 Cache Table – Duplicate Processing Prevention
- Prevents repeated verification of cartridges.
- Stores recently verified records for 30 days.
- Snapshot compared with cache; skips verification for duplicates.
- Records older than 30 days removed for periodic re-verification.

#### 3.3.3 Cartridge Identification Logic
- Unique hash identifier for each cartridge (product number, serial, manufacturer date, install date).
- Used to detect duplicates and track verification status.

#### 3.3.4 OCV Service Verification
- Records sent to external OCV verification service (AWS).
- Parallel batch processing (~100 records per batch).
- Checks: digital signature, authenticity, serial reuse, firmware compatibility.
- Returns: authentic, suspicious, invalid, or error.

#### 3.3.5 Export Table – Cartridge-Level Transformation
- Results stored in Export table.
- Expands printer records to cartridge-level.
- Filters duplicates before final results.

#### 3.3.6 History Table – Long-Term Storage
- Maintains historical record of cartridge authentication.
- Avoids duplicate entries unless status changes or new cartridge appears.
- Used by downstream analytics, reporting, warranty validation.

### 3.4 KLO Monitoring and Operational Checks
- Tracks health and execution status of ETL pipelines.
- Monitors 23 ETL jobs (14 Standard Raw, 9 OCV).
- Dashboards track job status, execution duration, metrics.
- Alerts triggered for failures or delays.

### 3.5 Data Volume Monitoring and Anomaly Detection
- Tracks daily data volumes for anomalies.
- Metrics: telemetry records processed, distinct printers.
- Compares with historical trends for spikes/drops.
- Weekly comparisons; persistent deviations investigated.

### 3.6 Seasonal Trend Analysis
- Accounts for expected seasonal variations (holidays, regional events).
- Uses historical trends to avoid unnecessary alerts.

### 3.7 Monitoring Tools and Automation
- Historically used Splunk dashboards.
- Enhancing with Databricks dashboards and alerts.
- Automates detection, notifications, abnormal pattern identification, alerts.

### 3.8 Data Retention Policies
- Different datasets have different retention policies.
- Ink telemetry retained longer; laser datasets shorter.
- Policies based on usage, regulations, cost.

---

## 4. Upstream/ Downstream details
- **Upstream:** Ingestion team
- **Downstream:** DST team

## 5. Production job schedules/Runbook
- Cumulus Prod Pipelines - Platform Big Data - HP R&D Wiki

## 6. Point of contact for failures
- **KLO:**
- KLO - Point of Contact for issues observed in data trends (Grafana) - Platform Big Data - HP R&D Wiki

---

## Images & Diagrams

### Telemetry Data Flow
![Telemetry Data Flow](image1.png)

### Source Table
![Source Table](image2.png)

---

## Source

| Source      | Point of Contact |
|-------------|------------------|
| Pepto       | Ludwig, David J <david.ludwig@hp.com>, Canino, Lawrence <lawrence.canino@hp.com> |
| WPPGENL     | Sajan, Reema Annie <reema.sajan@hp.com>, Bennadi, Bhavin (CW) <bhavin.bennadi@hp.com> |
| JAM         | Sraban Kumar Jena <sraban.kumar.jena@hp.com> |
| WIA         | Mandaknale, Deepak <deepak.mandaknale@hp.com> |
| WPPGEN2     | Gathman, Donald <don.gathman@hp.com>, Amdor, Diane <diane.amdor@hp.com> |
| Trident     | Verulkar, Anikesh <anikesh@hp.com> |
| Gotham      | Clements, Dan (CS R&A Data Orchestrator) <dan.clements@hp.com>, Nagarur, Ravindhranad <ravindhranad.nagarur@hp.com> |
| Printernet  | Padmanabhan, Anantha (CW) <anantha.padmanabhan@hp.com> |
|             | Packirisami, Swaminathan <swami@hp.com>, Mashru, Vipul <vipul.mashru@hp.com> |
|             | VISHNUMURTHY, RAVINDRA <ravindra.vishnumurthy@hp.com>, Malaka, Mohammed <mohammed.malaka@hp.com> |
|             | Jadhav, Ranjit Ramdas <ranjit.jadhav@hp.com>, sanjoy.bhattacharjee1@hp.com |
|             | Jim Marshall <james.marshall@hp.com> |
|             | Sharmila Padmanabhan <sharmila.padmanabhan@hp.com>, Kaviya, Kamala, Dhiren, Patil |
|             | suresh.kunder@hp.com, kamal.c.patel@hp.com |
