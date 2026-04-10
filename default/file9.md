Source tag : https://rndwiki.inc.hpicorp.net/confluence/spaces/BigData/pages/1506281442/OCV+Migration+to+DBT

Note : This extracted file is generated from SharePoint or Wiki content. If the source document contains images, charts, or graphical elements, they may not be fully converted into Markdown format. For complete clarity, please refer to the original source using the source tag provided above.

# OCV Migration to DBT

## Issue Description

This is an analysis requirement done on OCV Jobs for LASER sources for LEDM/MHIT formats for classic printer sources. The aim is to convert current code base of OCV being written in scala & python into dbt. This migrated dbt code will run on top of the SQL warehouses present in Databricks and we will ingest data into Unity Catalogs in Databricks instead of redshift tables.

## Analysis

The first part of the analysis involves in-depth understanding of OCV code at written in scala and python so that we can transition the same logic into dbt:

The Current OCV Job has in total 7 steps that run in sequence:

- 01-precheckmetrics
- 02-ocvprecheck
- 03-ocv
- 04-ocvbat
- 05-ocvloaderverifier
- 06-ocvloader
- 07-ocvloaderbat

### Step 01 - Pre-Check Metrics

Checks if the assumed role in Databricks DEV has "FULL_CONTROL" permissions over these objects (metrics/kibana_approved_only, metrics/pre_kibana, metrics/operations) under "hp-bigdata-databricks-prod-metrics-testresults" bucket.

### Step 02 - ocvprecheck

This step checks some of the pre-requisites and functionalities behaving properly.

1. Checks if the bonded-storage path is accessible. (This will be removed since we will be reading data from team_cumulus_dev.print_telem_private.laser_stdraw)
2. Checks if bonded storage has data.
3. Checks whether bonded-storage data has mvc 1 records. If yes then fails. If no then passes.
4. Checks whether we are able to connect to ocv service or not.
5. Checks if source_id present in cache table in databricks matches with the one given in parameter of the job.

### Step 03 - ocv

This is the main step in OCV Job where most of the important transformations and actions are performed:

Firstly if start_with_empty_cache is enabled it removes all the tables and database name as well.

Then it creates database and all the tables if they do not exist already in the below given sequence (All this is done in OCVTables class):

- **Database_name**: ocv_cache_wja_ledm_dpp
- **Cache_table**: ocv_cache_wja_ledm_dpp.cache_table
- **Export_table**: ocv_cache_wja_ledm_dpp.export_table
- **Drops Snapshot_table**: ocv_cache_wja_ledm_dpp.snapshots_table (This is done to drop snapshots table created in previous run so that new one can be created for this run.)

Then we refresh cache table to refresh the metadata for cache_tables to avoid any warnings whatsoever.

Then we remove cache_table_backup table. Then we refresh cache_table again. Then we purge expired entries for ocv_cache_wja_ledm_dpp.cache_table and remove records where lifetime of the records exceeds 30days in cache_table by seeing if the difference between current_date and ocv_lot_id column and if the difference is more than 30 then we remove the whole lot from the cache_table.

We remove records from cache_table that are older than 30 days. Get the pre and post counts of cache_table along with min and max insertion dates. Then we send this metrics to s3_metrics_output_path and s3_elk_outpu_path which is kibana metrics.

After purging expired entries we call OCVProcessing class which handles futher execution.

In OCV Processing class we save snapshots table in the database "ocv_cache_wja_ledm_dpp". Firstly it reads the bonded storage data for mvc 0 records only and then selects few columns from it and then filters out duplicate data based on hash column that it creates by concatenating ocv_id_hash and eea_valve_flag values. If the limit value given in params is not -1 then it limits the snapshots dataframe to the given value.

After reading the bonded storage data as a dataframe it creates a request_json column where it keeps all the required tags from consumable_config_dyn and product_config_dyn. It removes xml declarations from product_config_dyn so that it can be parsed. Then it removes ASCII Control Characters with ' ' (space). Then it creates an XML Element Tree object from the product_config_dyn. Then it finds the required tags in the xml_dyns and put the values in the required pcdTemplate which is again a xml template string. This gives us the prunedpDyn. Then we check for the missing serial number tag in product_config_dyn. If the serial_number tag is missing we create a tag in the dyn and assign the value that we got from the snapshots df.

For consumable_config_dyn the intial steps are same. But when it is creating the template it will look all the ConsumableInfo objects and in it ConsumableLabelCode objects (4 objects in total). If the value of ConsumableLable code objects is in the accepted values then it will create a template for that ConsumableInfo object with the required tags. Then it will append all the 4 templates with '\n' and create a final template by adding the ccheader and footer in it.

Then this prunedPDyn and prunedCDyn is appended in a dictionary of below format:

```json
{
  "uniqueid": {
    "data": {
      "LedmProductConfigDyn": "updatedPDyn",
      "LedmConsumableConfigDyn": "updatedCDyn"
    }
  }
}
```

This dictionary is dumped into a json string into the request_json column.

#### For LEDM Enterprises Sources

We do not prune either pdccfgdyn or ccfgdyn. We only add missing serial number tag to pdccfgdyn and append to the dictionary in above shown format and dump the json_string to request_json column.

#### For MHIT Sources

The above process was for LEDM sources. For MHIT sources there is just one single xml named complete_xml which is passed. For this we remove DeviceUsage and DeviceInformation tags. And then this xml string is appended in a dictionary of below format:

```json
{
  "uniqueid": {
    "data": {
      "RmaMhitUsageCollection": "mydyn"
    }
  }
}
```

This dictionary is dumped into a json string into the request_json column.

#### For PRINTERNET_MHIT_DPP Source

We have a seperate method that removes certain other columns and then appends this pruned xml into below given format:

```json
{
  "uniqueid": {
    "data": {
      "mhitDeviceService": "device_service_light",
      "mhitFimService": "fim_service",
      "mhitSuppliesService": "device_supplies_service"
    }
  }
}
```

Where:
- `mhitDeviceService`: This is the prunedXML
- `mhitFimService`: This is taken from fim_service_xml from snapshot
- `mhitSuppliesService`: This is taken from device_supplies_service_xml from snapshot

This dictionary is dumped into a json string into the request_json column.

#### For WJA_MHIT_DPP

In case of WJA_MHIT_DPP we don't have any xml to prune. So we create an XML temlate and fill in the data required from snapshot. Then this xml template converted to a xml string is fed to a dictionary in below format:

```json
{
  "uniqueid": {
    "data": {
      "RmaMhitUsageCollection": "wja_xml"
    }
  }
}
```

This dictionary is dumped into a json string into the request_json column.

Once request_json column is populated we drop certain non required columns from the snapshot. Get the count of total rows present in the snapshot. Save the snapshot as a table in databricks SQL named as ocv_cache_wja_ledm_dpp.snapshots_table as delta table. It then sets certain table properties and returns the count of rows in the snapshot.

### Processing Diagram

Below is the diagram representing further processing that happens in ocv step:

**Note**: The original document contains a complex flowchart diagram showing the OCV processing workflow. The diagram illustrates:

- Databricks Database structure (ocv_cache_jam_ledm_d)
- Table operations (cache_table, cache_table_backup, snapshots_table, export_table)
- Data flow through OCVProcessing.go() and OCVProcessing.doWorkThreaded()
- Authentication fields and OCV service interactions
- Request/response processing with chunking (500 chunks in total, 50000 records taken)
- Logic for cartridge authentication

Key components shown in the diagram:

1. **Database Tables**:
   - cache_table
   - cache_table_backup
   - snapshots_table (tmp table, dropped on every new job execution)
   - export_table

2. **Processing Steps**:
   - Refresh metadata for cache_tables
   - Drop cache_table_backup
   - Purge records older than 30 days from cache_table
   - Remove hash values from snapshots_table present in cache_table
   - Remove duplicates based on hash column
   - Create request_json format

3. **OCV Authentication Fields**:
   - Printer Authentication Result State (K/C/M/Y/OPC)
   - Online Authentication Result State (K/C/M/Y/OPC)
   - OCV Result Code (K/C/M/Y/OPC)
   - OCV Result Code Error
   - Cache Insertion Date

4. **Request Processing**:
   - Extract request_json value from each row
   - Compress each payload within 60 MB
   - Send request to OCV Service in chunks

5. **Logic for Cartridge Authentication**:
   - Check if supply has valid digital signature
   - Check if cartridge presents itself as HP
   - Check if cartridge passed low toner threshold on insertion
   - Check if printer has seen the cartridge before
   - Set Cartridge Life State accordingly (HP, Emulated, or Altered)

6. **Export Processing**:
   - Truncate already present export table
   - Copy records to export_table
   - Copies each of the supplies from cache_table to export_table
   - Resulting export_table should have 4X the records of cache_table (K/C/M/Y)

7. **History Table**:
   - Remove all records already present in history_table
   - Keep only new records
   - Track: Result Code, InPrinterCartridgeAuthentication, OnlineCartridgeAuthentication
   - Detect whether a supply has been seen in more than N different printer serial numbers

## Execution Timings

Execution timings of the PROD run vs DBT run:

| Steps | PROD | DBT |
|-------|------|-----|
| OCV main Notebook | 6m 42s | 7m 23s |
| Redshift load step | 15m 4s | 1m 26s |

The redshift load step time is reduced since we writing it to unity tables in databricks and not into redshift tables hence the reduction in time. We can achieve the same results without using dbt as well.

## Presentation for the OCV DBT Walkthrough

For analysis of moving OCV to DBT we created a mini POC of the OCV Job in DBT to run and capture the performance of the job.

### POC Approach and Observations

- We have segregated the OCV Code and created DBT models for all the query related transformations happening in the notebook.
- In our current OCV architecture we use parallel processing to send data in chunks to OCV Service. This use case was not applicable in DBT and hence we kept the main notebook and calling to OCV service part as it is in it's current state.
- On running the dbt job we observed similar run times compared to the current OCV job runs.
- Although the improvement is seen when writing the data to history table it is because we are writing to unity tables in Databricks instead of writing it to redshift.
- Other observational facts are that DBT job runs both job cluster and SQL compute in parallel incurring more cost and execution time is not reduced enough to cope for this extra cost.

### Final Decision

We had done a brainstorming session with the leads and considering all the above points the final decision was not go with DBT approach and just migrate to Unity.