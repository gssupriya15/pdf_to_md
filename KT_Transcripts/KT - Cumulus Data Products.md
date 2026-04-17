Source tag : https://hp.sharepoint.com/:w:/r/teams/DataOps310/Shared%20Documents/Data%20Ops%20documents/Documents/KT%20Transcript/KT%20-%20Cumulus%20Data%20Products.docx?d=wec78955e30d44ada9f90a6f1de3ef7bd&csf=1&web=1&e=GElLQU
Note : This extracted file is generated from SharePoint or Wiki content. If the source document contains images, charts, or graphical elements, they may not be fully converted into Markdown format. For complete clarity, please refer to the original source using the source tag provided above.

# KT - Cumulus Data Products

**Meeting Recording**
17 February 2026, 08:30am
1h 2m 16s

## Overview

Out of HP, we mostly work under print divisions. We are part of that organisation, TIO, specifically under Data OS. We handle telemetry data.

Telemetry data refers to printer-related data — number of pages printed, number of ink drops ejected (for ink printers), toner usage (for laser printers), printer idle time, and similar metrics. This data is captured as events or snapshots.

The primary business is supplies. We sell printers, but the main revenue comes from supplies. Based on telemetry data, we predict when customers are low on supplies and forecast demand in advance.

We process millions of records daily — terabytes of data on a daily basis. In addition to printer telemetry, we also collect app data and PC telemetry (though PC telemetry is handled by a separate team).

There are around 800 data products in total across various domains like supplies, subscription, engineering, marketing, and customer support.

## Data Sources

- Printers
- Apps
- PCs

## Ingestion Overview

There are two major data models:
- Legacy Data Model (LEDM)
- Common Data Model (CDM)

LEDM has been in use for 15+ years. CDM has been used for the last 3–5 years. Old printers still send data via LEDM, while new printers use CDM.

We handle HPS (Home Printing Services) and OPS (Office Printing Services), including SMB devices.

### LEDM Ingestion

Data models include LEDM, MIT, and AMPV. Data is ingested via multiple sources:
- Smart App
- Cartridge chip (pony Express)
- Enterprise printer systems
- Other drivers

The ingestion team (Pepto) receives the data into a landing bucket. They perform:
- Privacy tagging
- Geo enrichment
- Payload creation
- Archival storage

Privacy tagging ensures that opted-out customer data is filtered for analytics purposes.

## ETL (Cumulus Team)

We read data from Pepto and process it into Standard Raw (formerly Raw + Intermediate, now optimised). We have eliminated some intermediate storage for cost optimisation.

## CDM Flow

For CDM:
- Data flows into Jarvis Cloud Service
- Then into Stratus (telemetry service)
- Privacy tagging and deduplication occur
- Schema registry acts as source of truth
- Events are registered with schemas
- Task processing and enrichment pipelines run

CDM is event-driven (real-time). LEDM is snapshot-based (daily).

## Enterprise Data Product

Used for enrichment:
- Registration information
- Shipment and logistics
- Customer information
- Product sales
- Competitive sales

These are separate ingestion pipelines handled by another team.

## Analytics Layer

After ETL:
- Data flows into DST (IDST for ink, Trident for toner)
- Business logic and projections applied
- Data consumed by BDBT (Business Data & Media Team)

They generate:
- Dashboards
- Quarterly usage and share reports
- HP share calculations
- Install base and supply usage insights

## OCV (Online Cartridge Validation)

OCV validates whether cartridges are HP-authenticated. Data is sent to a web service, authenticated, and results are stored for downstream analytics.

## Data Quality

Standard Raw and OCV pipelines
~50 pipelines

Data Quality checks:
- Deduplication
- Record validation
- Product/serial validation
- Field rules and business rules
- Automated email reports
- Component back testing

Earlier used Splunk dashboards; now migrated to Databricks dashboards.

Operational metrics are written to Unity Catalog tables and used by Data Quality dashboards.

## Modernisation

- Migrated from Redshift to Delta Lake (Lakehouse)
- Unity Catalog adopted
- Storage reduced from ~800 TB to ~250 TB
- Serverless migration (Scala to Python)
- Pipelines converted to Python/PySpark

Automation agents under development for:
- Data product creation
- Job generation
- Monitoring
- KLO (Keep Lights On) automation

## Data Retention

- Standard Raw retains full history (since ~2003)
- CDM retains ~3 years
- XML “dines” retained for 5 years
- Non-dines retained fully

## Example Payload

Payloads are XML/JSON-like structures containing:
- Product configuration
- Product information
- Privacy objects
- Geo info
- Supply usage
- Multiple nested tags

Data lands in managed volumes (S3), read by ETL, parsed, validated, and written to Standard Raw tables.

## Privacy Handling

- Consent filtering
- Data valving
- Strictly necessary data tagging
- Filtering for analytics
- Encryption handled upstream (base64/zipped decoding during ETL)

## Current Focus

- Data quality enhancement
- Anomaly detection
- Pattern recognition in telemetry data
- Threshold-based alerts
- ML-based seasonal predictions
- Automating spike detection (e.g., privacy cache outage case causing 800k record spike)

## Future

- Pepto ingestion migrating to unified CDM-based Bronze++ layer
- LEDM payloads converted to CDM format via Retrobrush
- Single source of truth in Bronze++
- Standard Raw to consume from Bronze++

## Manual Checks

Mostly automated:
- Threshold alerts
- Failure-triggered pipeline stops
- Dashboard-based monitoring
- Mail alerts
- KLO automation agents in progress

## Closing

Thank you so much.
If you have more questions, please ping via email.

Thank you, everyone.
