Source tag : https://rndwiki.inc.hpicorp.net/confluence/spaces/BigData/pages/1756932400/Classic+ledm+new+pipeline

Note : This extracted file is generated from SharePoint or Wiki content. If the source document contains images, charts, or graphical elements, they may not be fully converted into Markdown format. For complete clarity, please refer to the original source using the source tag provided above.

# Classic ledm new pipeline

The Gotham Classic Ledm pipeline designed to ingest, decode, and transform Classic Ledm event data for downstream processing. This pipeline ensures compatibility with StdRaw by merging all relevant XMLs and metadata into a single payload, streamlining the data flow and maintaining legacy field conventions.

The decoding and payload construction logic is implemented as follows:

## Input Ingestion

The standalone process (gotham_classic_ledm) reads Classic Ledm events from the Unity Catalog table:
`team_onecloud_itg.cdm_bronze.domain_classicledm_v1_domain_ucdesoftware_v1_context_v1_privacy_v1`.

## Event Decoding

Each event is decoded using Base64, followed by Gzip decompression to retrieve the raw XML data.

## Data Extraction

The process extracts five XML fields from the event payload.
Privacy and Geo data are also extracted from the event structure.

## Payload Construction

All decoded XMLs, along with Geo and Privacy data, are merged into a single JSON payload using Spark DataFrame transformations.

## Consumable Type Identification

The pipeline identifies the consumable type (Ink or Laser) using the relevant enum.
Depending on the type:
- Ink payloads are written to Temporary Location A.
- Laser payloads are written to Temporary Location B.

## StdRaw Processing

StdRaw reads the payloads from the corresponding temporary locations and processes them for further actions.

## Pipeline Changes

### Job Configuration
Updates are made to databricks_job_config to support Ink and Laser StdRaw processing.

PR link :

### Airflow DAGs
A new task is added to cumulus-k8s-airflow2-dags for the new StdRaw pipeline.

PR link :

### Notebook Updates
The dps_stdraw now includes a new notebook (fetch_bronz) dedicated to event decoding and data extraction. The notebook merges all XMLs and metadata into a single payload for StdRaw compatibility.

### Metadata and Field Naming
Legacy field names are retained for compatibility. Conditional logic is added:

- If source_id = Gotham-hpx-ledm, set to Gotham_ledm.
- If pipeline_id = sage, set to pepto.

### Example Code Reference

PR link :