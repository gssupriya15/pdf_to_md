Source tag : https://rndwiki.inc.hpicorp.net/confluence/spaces/BigData/pages/1366289796/Pepto+-+Cumulus+Information+Page

Note : This extracted file is generated from SharePoint or Wiki content. If the source document contains images, charts, or graphical elements, they may not be fully converted into Markdown format. For complete clarity, please refer to the original source using the source tag provided above.

# Pepto - Cumulus Information Page

This page is to post information related to Pepto that is used by Cumulus.

Usually, Pepto issues are related to the following, which we started communicating in Data Products KLO Teams Channel, off late:

- **Missing hours** – Sometimes data arrives late. By the time our pipeline triggers, it wouldn't be available. Once data is recovered in pepto paths, we re-process all pipelines for that source manually for the recovered hour.

- **Low Data Ingestion for sources** – Off late, it is observed that for few sources data ingestion is low because of Pepto migration (23-Mar-2023 – low data ingestion), as posted in Data Products KLO Teams Channel

- **EEA_VALVE_FLAG** – Records whose eea_valve_flag should be set to false is set to true. Because of which data flows into our storage. This is due to Pepto Configuration changes related to eea_valve_flag.

- **Dynamic Privacy Object** - Pepto allocates dynamic privacy object for each record based on certain parameters. As on today, Cumulus has no way of verifying if dynamic privacy object is set or there is a failure in the process of setting it which leads to the application of static privacy object. If there would be a mechanism of letting Pepto know about the process success / failure, it would benefit the system. This scenario occurred on 25-April, 2023 when due to cache failure, Pepto could not append dynamic PO to data.

- **Pepto Alert Mechanism** - Proposed alert mechanism to see if data is written properly into Prod and Dev paths of pepto that are consumed by Cumulus. On 16-May-2023, there was an incident where write failure occurred for gotham in its prod path: `s3a://cscranalytics-dev-scratch/rawstore_mfio_v5/prod/mq/v5/1A01/prod/53989DE4A4426F820946434B/data/dev_detail_ledm` prod paths. If there was some mechanism to alert Pepto of the write failure, time could have been saved.

## Pepto Information from last 2 quarters (Nov-Jan, Feb-Apr)

### Missing Hours

| Missing Hours | Receive_Date | Source | Resolution Date | Jira Ticket |
|---------------|--------------|---------|-----------------|-------------|
| Hour - 21 | 14-Jan-2023 | WJA-MHIT | 04-Feb-2023 | [DOSDP-56226](https://hp-jira.external.hp.com/browse/DOSDP-56226) |
| Hour - 01 | 16-Jan-2023 | WJA-MHIT | 04-Feb-2023 | [DOSDP-56226](https://hp-jira.external.hp.com/browse/DOSDP-56226) |
| Hours – 07, 08, 09 | 22-Mar-2023 | WPPGEN1 | 23-Mar-2023 | Issue was communicated in Data Products KLO Teams Channel |
| Hours – 02-23 | 17-Mar-2023 | SAMSUNG | 17-Mar-2023 | Samsung source was decommissioned from Pepto end |

Above missing hours were not recovered from Pepto, hence we could not re-process from Cumulus End.

### Low Data ingestion, eea_valve_flag related:

| Jira Ticket | Issue Description | Resolution Date | Communication Channel |
|-------------|-------------------|-----------------|----------------------|
| [DOSDP-57666](https://hp-jira.external.hp.com/browse/DOSDP-57666) | Low Data Ingestion for 3 Ink, 7 Laser sources on 23-Mar-2023 | 04-April-2023 | Data Products KLO Teams Channel |
| [DOSDP-57665](https://hp-jira.external.hp.com/browse/DOSDP-57665) | Low Data ingestion for WJA, JAM sources on 27-Mar-2023 | 29-Mar-2023 | Data Products KLO Teams Channel |
| [DOSDP-57860](https://hp-jira.external.hp.com/browse/DOSDP-57860) | Low data ingestion for many sources since 07-Apr-2023 | Not Yet | Posted today in Data Products KLO Teams Channel |
| [DOSDP-57667](https://hp-jira.external.hp.com/browse/DOSDP-57667) | Eea_valve_flag is set to true instead of false for EU Records | Not Yet | Data Products KLO Teams Channel. Update is tracked in Jira Ticket |