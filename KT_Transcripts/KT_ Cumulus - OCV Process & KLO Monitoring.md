Source tag : https://hp.sharepoint.com/:w:/r/teams/DataOps310/Shared%20Documents/Data%20Ops%20documents/Documents/KT%20Transcript/KT_%20Cumulus%20-%20OCV%20Process%20%26%20KLO%20Monitoring.docx?d=wda143185f11a44d1ae5be1380d8f15db&csf=1&web=1&e=vrgsuM

Note : This extracted file is generated from SharePoint or Wiki content. If the source document contains images, charts, or graphical elements, they may not be fully converted into Markdown format. For complete clarity, please refer to the original source using the source tag provided above.

# KT Cumulus - OCV Process & KLO Monitoring

**Meeting Recording**

February 25, 2026, 5:30AM  
Duration: 53m 37s

---

### Introduction

1:20  
Yeah.  
Okay, shall I start? My screen is visible right to everyone.

1:37  
Yeah, yeah, you can start.

1:41  
Mm.  
So, I've created this wiki regarding to OCV long back, so I'll be referring to this only, and we'll go through it.

So after standard raw pipeline comes the OCV pipeline. OCV takes the data from standard raw. Whatever standard raw has dumped in laser standard raw and standard raw jobs, OCV takes data from laser standard raw.

OCV stands for **Online Cartridge Verification**.

The purpose of this pipeline is to verify the cartridges that the printers have installed in them. This pipeline is only for laser sources, not for ink sources currently.

We have around 9 pipelines running for laser sources (excluding one source).

These are the notebooks that run in the OCV pipeline daily for all nine sources:

- Pre-check metrics (checks table access)
- OCV pre-check (validates conditions)
- Main OCV processing notebook

In the main processing notebook, we call an OCV service API hosted in AWS.

We send data in chunks of 100 rows, and the service returns results for each row.
Each row represents a single printer.
The service returns additional columns that indicate whether the printer has an authentic cartridge installed or not.

---

### Q&A and Technical Details

**6:04**  
I just have a question. What criteria are used to identify duplicates? Is it the printer ID?

**6:06**  
Duplicates are identified using a hash column.

**6:23**  
Okay.

**6:26**  
We concatenate several columns to create an OCV hash.
A printer can have 4 cartridges (K, C, M, Y).
So we generate 4 hash values, one for each cartridge.
These are then concatenated with other fields to produce a final hash which acts as the unique ID for each printer.

**7:27**  
Noted, thanks.

Another table involved is the **snapshot table**, which is a copy of the standard raw table but contains only the columns needed for processing.

We also have:
- Cache table
- History table

The cache table helps reduce processing.

*Example:*  
If we receive 100 records, many might already be processed in the past 30 days.  
So we check the cache table and process only the new records.  
The cache table stores only 30 days of data.  
Older records are purged every day.

**10:36**  
So the snapshot table contains only daily data?

**10:37**  
Yes. Snapshot is temporary and contains only the latest data from standard raw.  
It is recreated every day.

**11:22**  
So snapshot has today's data and cache has last 30 days of data?

**11:29**  
Yes.

**11:38**  
If the same printer reports daily, wouldn't it repeat?

**11:53**  
Printers report daily, but cartridges usually do not change daily.  
So a printer is checked again only after 30 days unless cartridge information changes.

**13:10**  
Is KCMY related to cartridge serial numbers?

**13:13**  
KCMY are color codes:
- K = Black
- C = Cyan
- M = Magenta
- Y = Yellow

Each printer can have 4 cartridge records.

---

### Processing Details

**15:38**  
Once records are identified, we process them using 5 parallel threads.

*Example:*  
- 50,000 records
- Split into chunks of 100 records
- ~500 chunks created
- 5 chunks processed simultaneously

Each chunk is converted into a JSON payload and sent to the OCV API.

**16:04**  
Is the service hosted on EC2?

**16:10**  
It is hosted in AWS, possibly using Lambda.  
It is maintained by a different team.

**16:36**  
Are we using SNS or SQS?

**16:36**  
No. Threading is implemented directly in the code to send API calls in parallel.

The OCV service validates cartridges using several checks:
- Whether the printer has seen the cartridge before
- Digital signature validation
- Cartridge authenticity checks
- Serial number duplication across printers

It returns:
- Result Code
- Printer Cartridge Authentication Result
- Online Cartridge Authentication Result

---

### Data Storage and Export

**20:44**  
After processing, results are stored in the cache table.  
The process loops until snapshot table becomes empty.  
Then the data is moved to an export table.

In the snapshot table:
- 1 printer = 1 row (with KCMY columns)

In export table:
- 1 printer = multiple rows (one per cartridge).

Duplicates are removed here as well.  
Only records where authentication state changes are pushed to the history table.

**22:02**  
Is the history table used by another team?

**22:02**  
Yes, it is used for warranty-related purposes.

**23:08**  
What about privacy concerns if the cartridge is from another manufacturer?

**23:13**  
Those records are still stored.  
The history table simply records whether authentication passed or failed.

**24:25**  
Privacy filtering already happens in the standard raw pipeline, so OCV doesn't need to apply additional filtering.

---

### ETL Perspective

**29:30**  
From an ETL perspective:

Final persisted tables are:
- Cache table
- History table

Other tables like snapshot and export are temporary.

---

## KLO Monitoring

**32:48**  
Daily monitoring is done to check whether all pipeline jobs succeed.

Total jobs monitored: 23

Breakdown:
- Standard Raw Pipeline – 14 jobs
- Ink: 4 jobs
- Laser: 10 jobs
- OCV Pipeline – 9 jobs

A dashboard is used to check job success.

*Example:*  
For a specific date, we verify whether all 23 jobs succeeded.

**33:11**  
Another monitoring aspect is data count validation.  
Daily counts are compared to detect anomalies.

Example metrics monitored:
- Total record count
- Distinct printer count

If there are significant drops, the integration team is notified.

**35:48**  
Is this monitoring manual?

**35:59**  
Previously alerts were configured in Splunk.  
Currently dashboards are being replicated in Databricks, and automated alerts to Teams channels are being implemented.

**37:42**  
Deviation thresholds:
- Alerts trigger when deviation exceeds ±5% for more than 3 days.
- Some deviations occur due to seasonal patterns (e.g., holidays or regional events).
- An ML-based seasonal trend dashboard helps identify such cases.

**41:05**  
The team is also exploring using agents for anomaly detection in monitoring dashboards.

---

### Retention Policy & Streaming

**47:41**  
Retention policies discussed:
- Ink data: 5 years
- Laser data: 2 years
- For some pipelines, full historical data is maintained.

**49:34**  
Streaming pipelines for some components are already implemented and support near real-time processing.

**51:10**  
Retention policy depends on:
- Data criticality
- Legal requirements
- Business usage

---

### Tools Used

**53:00**  
Tools used:
- Databricks
- Wiki
- GitHub

**53:21**  
Thank you.

**53:27**  
Thank you.

**53:28**  
Thank you.

**53:31**  
Bye.
