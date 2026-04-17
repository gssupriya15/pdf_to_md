Source tag : https://hp.sharepoint.com/:w:/r/teams/DataOps310/Shared%20Documents/Data%20Ops%20documents/Documents/Understanding%20docs/Cumulus_Data_Product%20-%20overview.docx?d=wa16fc5b403f44b698146fc1d29bfe577&csf=1&web=1&e=8hf7Lu

Note : This extracted file is generated from SharePoint or Wiki content. If the source document contains images, charts, or graphical elements, they may not be fully converted into Markdown format. For complete clarity, please refer to the original source using the source tag provided above.

# Cumulus Data Products

This document provides a detailed explanation of the Cumulus Data Product within HP’s DataOS ecosystem. The content will explain the architecture, data flow, processing layers, and data quality mechanisms discussed during the Knowledge Transfer session.

## 1. Overview of HP DataOS and Data Products

- HP DataOS is a centralized data ecosystem that manages and processes large volumes of telemetry and enterprise data.
- Within this ecosystem, there are approximately 800 data products. Each data product is designed to serve a specific business purpose. Cumulus is one such data product, primarily focused on printer telemetry data for the Supplies business.
- HP printers continuously generate telemetry data such as pages printed, ink drops ejected, toner usage, device status, error logs, and operational metrics.
- This data is transmitted to cloud services through firmware and driver mechanisms.
- Telemetry data is collected from various sources including home printers, office printers, enterprise printers, smart applications, and related systems.

## 2. Architecture

We have 3 major layers present in Data OS:

1. Ingestion
2. ETL (Cumulus)
3. DST

## 3. Ingestion Layer

There are two types of ingestion in Data OS.

### 3.1 The Legacy Data Model (LEDM)

- Captures printer data in snapshot format at specific points in time, typically using XML-based payloads.
- Legacy data ingestion is done by Pepto.
- **Models:** LEDM, MHIT, AMPV
- **Sources:** Pony Express, Gotham, JAM

### 3.2 The Common Data Model (CDM)

- A modern, event-driven model where printer events are captured in real time.
- CDM uses schema registry as the source of truth and follows standardized event structures.
- **Sources:** PSDR, Smart app, Smart driver

**Overall Process:**

- Telemetry data first lands in a landing bucket managed by the ingestion team.
- Privacy tagging and consent validation are applied before the data is forwarded for further processing. If a customer has opted out of data usage for analytics or marketing, the system ensures such data is filtered appropriately. Privacy compliance is handled at multiple layers to ensure regulatory adherence.
- Along with privacy tagging, some geo information is added and a payload will be created. This payload will be put into an archival bucket.
- Both Pepto and Bronze++ have similar processes of data ingestion. The difference is in the schema; tables will be different for both.

## 4. ETL Layer (Cumulus)

Cumulus functions as the ETL layer within DataOS. It reads data from the ingestion layer for both CDM and LEDM and performs the following operations:

- Removing duplicates
- Validates product identifiers
- Apply some transformation rules
- Extracts required attributes from payloads
- Archival storing

### 4.1 LEDM

- Once the data from Pepto is read, it will be processed into STDRAW.
- The processed data is structured and stored in Delta Lake tables using Unity Catalog.
- Previously, intermediate raw storage existed, but cost optimizations have streamlined the architecture.

### 4.2 CDM

- Data is read from Bronze++ and will be loaded to PSU (Print Supply Usage)

#### OCV – Online Cartridge Validation

- OCV is a separate validation pipeline that verifies whether cartridges used in printers are genuine HP cartridges. This is a continuation of LEDM pipeline after the STDRAW process.
- The pipeline communicates with a validation service and stores authenticated results. This validation plays a crucial role in business analytics, particularly in calculating usage share and HP supply market share.

## 5. DST Layer

- Data from STDRAW and PSU combined will be landed into another layer called DST where processing related to analytics will be performed.
- Within DST we have IDST which is for Ink and TRIDENT is for toner/laser.
- In other words, ETL layer focuses on data flattening, cleansing will be done. Whereas in DST layer the aggregations, enriching data based on business logic/use.
- Once the data is processed in DST, it will be used by BDBT team for analytics. BDBT is the downstream which focuses on the supplies. They have several dashboards on Thoughtspot for analysis. These insights will be used to derive usage and shares.

## 6. Enterprise Data Enrichment

Printer telemetry data is enriched with enterprise data such as shipment details, customer registration, geolocation, product family information, and logistics data. This enrichment helps derive meaningful business insights such as install base, usage trends, forecasting, and quarterly market share reporting.

## 7. Data Quality and Monitoring

- Data quality checks are embedded within ingestion pipelines.
- Automated validation rules verify the following:
  - record counts
  - schema consistency
  - threshold limits
  - business logic compliance
- Operational metrics are logged and stored in Unity Catalog tables.
- There are ML models which predict seasonal trends.
- There is a separate quality team which monitors the entire data quality from ingestion, ETL, DST and other teams. They will read it from delta tables which have data quality information.

## 8. Archival Strategy and Data Retention

- Historical data is retained for compliance and analytics. Older data may be moved to archival storage to reduce cost while preserving accessibility for audit and long-term analysis.
- STDRAW has data from 2003 which initially had 2-3 sources and now it has 13 sources.
- CDM retains data for 3 years.
- In STDRAW we have dyns and non-dyns which are nothing but the xml tags.
- Dyns has retention policy for 5 years. Laser dyns has 3 years of data retention policy.
- Non-dyns data doesn’t have any retention policy; everything is retained.

## 9. Modernization and Future Enhancements

- The organization is migrating pipelines to serverless environments (already migrated from the scala code to pyspark) and standardizing ingestion through unified data models.
- From ingestion side, there is a plan to migrate the Pepto to CDM model. The process involves converting the LEDM payload into CDM ones and creating a single source of truth.
- Automation initiatives include data product generation agents and enhanced anomaly detection mechanisms. The long-term vision focuses on improved data quality intelligence and automated monitoring across the entire telemetry ecosystem.

## Conclusion

Cumulus plays a critical role in transforming raw printer telemetry data into structured, business-ready datasets that drive forecasting, market analysis, and operational insights. Through continuous modernization and automation efforts, HP aims to enhance scalability, cost efficiency, and data reliability within the DataOS ecosystem.
