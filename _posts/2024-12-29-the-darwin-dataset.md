# Designing a System for National Rail's Darwin Push Port Feed

Over the past few months, I have been designing a system for ingesting data from National Rail’s “Darwin Push Port Feed”. This feed contains live information about train movements, scheduling, and service alterations across the UK rail network. In this article, I want to walk through the final structure of the data so any new users can quickly understand and work with the dataset.

## Overview of the Dataset

The dataset contains two separate streams of data, stored in S3 in parquet format. One stream encodes information about **services**, and the other about **locations**. While the data received from the Darwin feed is heavily nested JSON, the aim of this dataset was to split up entities as much as possible to make it easy to query via SQL. On ingestion into the system, each message gets parsed and split into multiple rows in each table.

---

## The Service Table

The **services** table encodes information about the service itself, such as identifiers, timestamps, and other metadata. For each message received, a new entry is made in the service table capturing the listed metadata from the message. This means each service row is more accurately described as a snapshot of the metadata for a service at a given moment.

> **Note**: Different message types may contain different metadata, which can cause inconsistencies.

### Schema for the Service Table

| Column Name | Datatype | Message Type  | Comment                                                     |
| ----------- | -------- | ------------- | ----------------------------------------------------------- |
| rid         | string   | All messages  | The unique identifier for a service. Given as YYYYMMDD{uid} |
| uid         | string   | All messages  | A unique ID for a service on a given day                    |
| ts          | string   | All messages  | The timestamp of the message on ingestion                   |
| passenger   | boolean  | Schedule only | Denotes if a service is a passenger service or not          |
| update_id   | string   | All messages  | A UUID identifying the unique ID of this message            |
| toc         | string   | Schedule only | The identifier for the train operating company              |
| train_id    | string   | Schedule only | The headcode of a given service                             |
| year        | string   | All messages  | The year the message was ingested                           |
| month       | string   | All messages  | The month the message was ingested                          |
| day         | string   | All messages  | The day the message was ingested                            |

---

## The Locations Table

The **locations** table contains individual records for each stop a service is scheduled, estimated, or makes over the course of a journey. These rows are joined together by the `service_update_id`, which corresponds to the `update_id` from the service table.

### Schema for the Locations Table

| Column Name       | Datatype | Comment                                                                                      |
| ----------------- | -------- | -------------------------------------------------------------------------------------------- |
| tpl               | string   | The TPL location code for the location                                                       |
| type              | string   | Encodes either ARR/DEP                                                                       |
| time_type         | string   | Encodes either: SCHED (Scheduled), EST (Estimated), or ACT (Actual) timings for the location |
| time              | string   | The time of the location, encoded as HH:MM:SS                                                |
| location_id       | string   | UUID uniquely identifying the location                                                       |
| service_update_id | string   | The service update the location was from (Foreign Key for service `update_id`)               |
| year              | string   | The year the message was ingested                                                            |
| month             | string   | The month the message was ingested                                                           |
| day               | string   | The day the message was ingested                                                             |

---

## Query Examples

### Retrieve All Service IDs for a Given Day

```sql
SELECT rid, uid
FROM services_v1
WHERE year = '2024'
  AND month = '12'
  AND day = '28'
GROUP BY rid, uid;
```

### Retrieve All Locations for a Specific Service ID

```sql
SELECT tpl, type, time_type, time
FROM services_v1
LEFT JOIN locations_v1 
  ON services_v1.update_id = locations_v1.service_update_id
WHERE services_v1.year = '2024'
  AND services_v1.month = '12'
  AND services_v1.day = '28'
  AND services_v1.rid = '202412288013192'
ORDER BY time;
```

## Appendix: Data Indexing and Partitioning

To run these queries, I have indexed the data via AWS Glue using standard Glue crawlers. This created the two tables described above. You can do the same!

Tip: The data is partitioned in Hive style using year, month, and day. Using these partitions in your queries will drastically reduce query costs. Learn more here.