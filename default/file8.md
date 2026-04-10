Source tag : https://rndwiki.inc.hpicorp.net/confluence/spaces/BigData/pages/1633028032/Cumulus+Data+Retention

Note : This extracted file is generated from SharePoint or Wiki content. If the source document contains images, charts, or graphical elements, they may not be fully converted into Markdown format. For complete clarity, please refer to the original source using the source tag provided above.

# Cumulus Data Retention

| Data Product | Account | Path | Policy |
|--------------|---------|------|--------|
| ink_raw_ledm | bigdata-itg | s3://hp-bigdata-databricks-prod/ink-raw-delta-lake/LEDM | Archive data older than 3 months goes to Glacier (due to legal hold) |
| laser_raw_ledm | bigdata-itg | s3://hp-bigdata-databricks-prod/laser-raw-delta-lake/LEDM | Archive data older than 3 months goes to Glacier (due to legal hold) |
| laser_raw_mhit | bigdata-itg | s3://hp-bigdata-databricks-prod/laser-raw-delta-lake/MHIT | Archive data older than 3 months goes to Glacier (due to legal hold) |
| laser_raw_ampv | bigdata-itg | s3://hp-bigdata-databricks-prod/laser-raw-delta-lake/AMPV | Archive data older than 3 months goes to Glacier (due to legal hold) |
| ink_stdraw | dataos-core-prod-team-cumulus | s3://dataos-core-prod-team-cumulus/catalog/__unitystorage/catalogs/13c596da-b3f9-40ad-8cf7-77cea9e00b6e/tables/13af1a8a-e2e3-4b0f-aa6b-cf0839c7b79b | All data retained currently. |
| ink_stdraw_dyns | dataos-core-prod-team-cumulus | s3://dataos-core-prod-team-cumulus/catalog/__unitystorage/catalogs/13c596da-b3f9-40ad-8cf7-77cea9e00b6e/tables/8440e5ac-b023-41eb-9de2-6da37f037067 | All data retained currently. |
| laser_stdraw | bigdata-itg | s3://hp-bigdata-databricks-prod/laser-stdraw-data-lake-delta | All data retained currently. |