Source tag : https://hp.sharepoint.com/:w:/r/teams/DataOps310/Shared%20Documents/Data%20Ops%20documents/Documents/KT%20Transcript/KT_%20Cumulus%20-%20ETL%20StdRaw%20Process.docx?d=w3fd869b2dd39429fa955a54650d15956&csf=1&web=1&e=IVWoGM
Note : This extracted file is generated from SharePoint or Wiki content. If the source document contains images, charts, or graphical elements, they may not be fully converted into Markdown format. For complete clarity, please refer to the original source using the source tag provided above.

# KT – Cumulus – ETL StdRaw Process

**Meeting Recording:** 20 February 2026, 05:30am
**Duration:** 38m 42s

## Session Introduction

2:23
Okay, so Aditya is with me today. In the last session, we discussed the overall data flow and architecture. Today, he will walk you through the Standard Raw (StdRaw) ETL process at a high level—not deep code, but architecture and logic.

2:49
Yeah, thanks. We can start.

## StdRaw Overview

To begin with, we are the team handling LMS sources and related telemetry data. The main source for StdRaw is the Pepto landing layer, from where we fetch payload data.

We manage three major data products:
- PUCJ
- OCV
- Xsell

The StdRaw pipelines run daily.
- Total daily data processed: ~700 GB
- Overall runtime: ~3 hours
- Total jobs: 14

Out of these:
- 4 jobs for Ink sources
- 10 jobs for Laser sources

Sources include:
- Pony Express
- Gen2
- LMS
- MID
- AMPV

All payloads are read from Pepto.

## Job Structure

We have:
- One notebook for Ink
- One notebook for Laser

Each notebook is reused across multiple source IDs. The metadata drives the source-specific behavior.

Each pipeline run consists of three major components:
1. Main Body
2. BAT
3. Component BAT

These are executed sequentially within the same job.

### 1️⃣ Main Body (Core ETL)

The main body performs:
- Data extraction
- Transformation
- Loading

We read payloads based on:
- Source ID
- received_dt (T-1 data)

The payload is:
- Nested JSON
- Contains embedded XML
- Stored as string in Unity Catalog

Schema differs per source. It is not static.

We extract:
- ~800 columns for Ink
- ~1200+ columns for Laser

#### Metadata-Driven Framework

All column extraction logic is driven by a metadata Excel sheet stored in Unity Volume.

Metadata includes:
- XML extraction path
- PySpark extraction expression
- Transformation recipe
- Cleansing logic
- Source-specific rules

It is:
- Separate metadata file for Ink
- Separate metadata file for Laser
- Within file, logic separated by source

Metadata updates happen only when:
- Business requests new counters
- Logic changes for existing counters

Not frequent.

#### Transformations & Validations

Main body also handles:
- Deduplication
- Filtering malformed records
- Type casting
- Cleansing
- Required field validation
- Error code mapping
- Flag generation

Out of 1200 columns, ~15–20 are critical.

If:
- Critical field fails → Job fails
- Non-critical issue → Flagged with warnings

Flags:
- 0 → Invalid record
- 1 → Valid record

Master validity code determines whether record is forwarded.

Malformed records:
- Flagged during processing
- Filtered before final load

#### Dines vs Non-Dines Tables

We split target tables into:

**Non-Dines**
- Frequently accessed columns
- Used heavily by downstream teams

**Dines**
- XML-heavy columns
- Rarely accessed

Separate tables created in Unity Catalog:
- ink_standard_raw
- ink_standard_raw_dines
- laser_standard_raw
- laser_standard_raw_dines

#### Storage Optimization

Historical data since 2003.

Earlier:
- Stored as Parquet in S3
- ~800 TB storage

After optimization:
- Migrated to Unity Catalog
- Converted to Liquid Clustering
- Applied ZSTD compression
- Lifecycle policies on Dines

Current storage:
- ~250 TB

Retention:
- Laser Dines → Last 2 years
- Ink Dines → Last 5 years

Significant cost savings achieved.

### 2️⃣ BAT (Data Profiling Layer)

BAT runs immediately after main body.

Purpose:
- Daily data profiling
- Validation metrics
- Summary reporting

Generates email report including:
- Source vs target count validation
- Good vs bad count
- Null count & percentage
- Duplicate check
- Rule validation status
- Recipe application check

Implemented using:
- PySpark
- JSON generation
- HTML formatting for email

No external framework used.

Runs:
- Daily
- Only on current day data
- As part of same job

Status:
- Pass
- Pass with Warning
- Fail

If Fail:
- Pipeline fails at BAT stage
- Data already loaded by main body
- May revert using Delta versioning

### 3️⃣ Component BAT

This performs post-load validation.

Key aspects:
- Runs on 25–50% sample
- Re-validates transformations
- Re-checks metadata-driven logic
- Validates derived columns
- Validates error lists
- Validates master validity code
- Ensures OCV-related hashes are not null

If:
- Critical failure → Pipeline fails
- Warning → Logged and monitored

Data already loaded in table; if needed:
- Overwrite
- Or revert using Delta Lake version

## Parallelism

All 14 jobs:
- Run in parallel
- No inter-source dependency

Delta Lake supports:
- Parallel writes to same table

Total runtime:
- < 3 hours

## Failure Handling

Failures are mostly:
- Data issues from source
- Firmware updates causing malformed data
- Missing data
- Intermittent Databricks executor issues

Rarely:
- Code-level issue

Resolution:
- Source team re-triggers
- We re-run pipeline
- Or revert via Delta version

## KLO Agent (Automation)

KLO monitoring agent:

If:
- Intermittent failure → Auto re-trigger
- No data available → Notification to source team
- Code issue → Manual intervention

Currently:
- Used more heavily by enterprise teams
- StdRaw exploring advanced anomaly detection

## Future Focus

Team wants to move toward:
- Pattern recognition in incoming payloads
- Column-level anomaly detection
- Data drift detection
- Agent-based deviation monitoring

Goal:
Move beyond rule-based checks to intelligent quality detection.

## Monitoring

Dashboards exist for:
- Job monitoring
- Runtime metrics
- Validation tracking

To be demonstrated in next session (OCV KT).

38:21
Wiki access will be provided.
Please share names and emails.

38:36
Thank you, everyone.
