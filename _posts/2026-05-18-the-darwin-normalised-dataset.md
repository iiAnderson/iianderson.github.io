---
layout: post
title: "The Darwin Normalised Dataset: one row per train, every day"
categories: [Railways, Data]
tags: [railway, dataset, aws, open-data, darwin, sql, documentation]
---

Since late 2024, I have been collecting real-time data from National Rail's Darwin Push Port feed — every schedule update, every delay, every cancellation — storing it as Parquet files on S3. The raw data now exceeds 600 million service rows and nearly 4 billion location rows, covering every train movement on the UK rail network.

The raw tables are powerful, but they are not friendly. Each train service generates multiple messages as its schedule is updated throughout the day, meaning you need to deduplicate across messages, join services to locations, handle sparse fields that only appear in some messages, and account for midnight crossings. A simple question like "was this train cancelled?" requires checking every message for that service and correctly resolving reinstatements.

To make this data accessible, I have built **normalised_v1** — a single, analysis-ready table with **one row per train service**. Delays are pre-computed, cancellation status is resolved, and every calling point is packed into a nested array you can unnest in a single query. The dataset is now available on the [AWS Open Data Registry](https://registry.opendata.aws/), and this post is the reference guide for using it.

Whether you want to study cancellations, delays, overcrowding, or just explore what trains ran on a given day — this dataset is designed to get you from question to answer in a single query.

## What is normalised_v1?

Each row in `normalised_v1` represents one train service on the UK rail network — approximately **30,000 services per day**. The data is produced daily at 4 AM UTC from the raw Darwin data, covering the previous day's services.

Key facts:

- **Format**: Hive-partitioned Apache Parquet (Snappy compressed)
- **S3 path**: `s3://darwin-connect/normalised/v1/year=YYYY/month=MM/day=DD/data.parquet`
- **Region**: `eu-west-1`
- **Requester Pays**: Yes — you pay for data transfer when querying
- **Partitioned by**: Service date (the date the train was scheduled to run), not the date messages were ingested
- **Data available from**: 1 January 2025 onwards
- **Known gaps**: 14 days in January 2025 and 10 days in February 2025 (ingestion outages)
- **Update frequency**: Daily — yesterday's services become available after 4 AM UTC today

## Getting started

The fastest way to query the dataset is with AWS Athena. Create the table with the following DDL:

```sql
CREATE EXTERNAL TABLE IF NOT EXISTS normalised_v1 (
  rid                       string,
  service_date              string,
  toc                       string,
  train_id                  string,
  passenger                 boolean,
  origin_tpl                string,
  origin_sched_dep          string,
  destination_tpl           string,
  destination_sched_arr     string,
  num_sched_stops           int,
  num_act_stops             int,
  cancellation_status       string,
  cancel_reason             string,
  delay_reason              string,
  cancellation_ts           string,
  minutes_before_origin_dep int,
  origin_delay_mins         int,
  destination_delay_mins    int,
  avg_delay_mins            double,
  max_delay_mins            int,
  stops_on_time             int,
  stops_minor_delay         int,
  stops_moderate_delay      int,
  stops_major_delay         int,
  avg_loading               double,
  loading_0_20              int,
  loading_20_40             int,
  loading_40_60             int,
  loading_60_80             int,
  loading_80_100            int,
  stops                     array<struct<tpl:string,arr_sched:string,dep_sched:string,arr_act:string,dep_act:string,arr_delay:int,dep_delay:int,cancelled:boolean>>
)
PARTITIONED BY (
  year  string,
  month string,
  day   string
)
STORED AS PARQUET
LOCATION 's3://darwin-connect/normalised/v1/'
TBLPROPERTIES ('parquet.compression'='SNAPPY');
```

Then discover all available partitions:

```sql
MSCK REPAIR TABLE normalised_v1;
```

Verify everything is working with a quick count:

```sql
SELECT COUNT(*) AS num_services
FROM normalised_v1
WHERE year = '2025' AND month = '10' AND day = '01';
```

You should see around 30,000 services for a typical weekday.

Athena is not your only option — the Parquet files work with any tool that reads Parquet. DuckDB, Apache Spark, pandas, and PyArrow can all read directly from S3.

## The schema

The table has 32 top-level columns plus a nested `stops` array. Here is the full reference, grouped by purpose.

### Service identification

| Column | Type | Description |
|--------|------|-------------|
| `rid` | string | Unique service identifier. Formatted as `YYYYMMDD{uid}` (e.g., `202510018013192`). The first 8 characters encode the service date. |
| `service_date` | string | Service date in `YYYYMMDD` format, extracted from `rid` for convenience. |

### Operator and type

| Column | Type | Description |
|--------|------|-------------|
| `toc` | string | Train operating company code (e.g., `GW` for Great Western Railway, `SE` for Southeastern, `VT` for Avanti West Coast). |
| `train_id` | string | Train headcode (e.g., `1A23`). |
| `passenger` | boolean | `true` for passenger services, `false` for empty stock movements and other non-passenger workings. |

### Route

| Column | Type | Description |
|--------|------|-------------|
| `origin_tpl` | string | TIPLOC code of the scheduled origin station (e.g., `BRSTLTM` for Bristol Temple Meads). |
| `origin_sched_dep` | string | Scheduled departure time at origin (`HH:MM:SS`). |
| `destination_tpl` | string | TIPLOC code of the scheduled destination. |
| `destination_sched_arr` | string | Scheduled arrival time at destination (`HH:MM:SS`). |

TIPLOC codes are internal identifiers used by the rail industry — see [Resolving station names](#resolving-station-names) below for how to map them to human-readable station names.

### Stop counts

| Column | Type | Description |
|--------|------|-------------|
| `num_sched_stops` | int | Number of scheduled stops (the full planned route, including any that were later cancelled). |
| `num_act_stops` | int | Number of stops where actual times were recorded (i.e., the train actually called). Zero for fully cancelled services. |

### Cancellation

| Column | Type | Description |
|--------|------|-------------|
| `cancellation_status` | string | One of `'ran'`, `'cancelled'`, or `'partially_cancelled'`. |
| `cancel_reason` | string | Cancellation reason text from National Rail. NULL if the service ran or was reinstated. |
| `delay_reason` | string | Delay reason text. |
| `cancellation_ts` | string | ISO 8601 timestamp of the earliest cancellation signal. NULL if the service ran. |
| `minutes_before_origin_dep` | int | Minutes between cancellation announcement and scheduled departure. Positive means announced before departure. NULL if the service ran. |

The three cancellation states mean:

- **`ran`**: The service completed its route (or at least departed — no cancellation signal was received).
- **`cancelled`**: The service never ran at all. No actual times were recorded at any stop.
- **`partially_cancelled`**: The train ran part of its route but was cancelled for some stops. At least one stop has actual times, but one or more stops were marked as cancelled.

Reinstatements are correctly resolved — if a service was initially cancelled but later reinstated, it will show as `ran`.

### Delay metrics

| Column | Type | Description |
|--------|------|-------------|
| `origin_delay_mins` | int | Delay at the origin in minutes. NULL if the origin was not reached. |
| `destination_delay_mins` | int | Delay at the destination in minutes. NULL if the destination was not reached. |
| `avg_delay_mins` | double | Average delay across all stops with both scheduled and actual times. |
| `max_delay_mins` | int | Maximum delay at any single stop. |

These figures already account for midnight crossings — if a service departs before midnight and arrives after, the delay is computed correctly. You do not need to handle this yourself.

### Delay categories

| Column | Type | Description |
|--------|------|-------------|
| `stops_on_time` | int | Number of stops where the delay was 0–5 minutes. |
| `stops_minor_delay` | int | Number of stops with a 5–15 minute delay. |
| `stops_moderate_delay` | int | Number of stops with a 15–30 minute delay. |
| `stops_major_delay` | int | Number of stops with a delay exceeding 30 minutes. |

These buckets are useful for quick classification without needing to unnest the stops array.

### Passenger loading

| Column | Type | Description |
|--------|------|-------------|
| `avg_loading` | double | Average passenger loading as a percentage (0–100) across all reported stops. |
| `loading_0_20` | int | Number of stops with loading under 20%. |
| `loading_20_40` | int | Stops with loading between 20% and 40%. |
| `loading_40_60` | int | Stops with loading between 40% and 60%. |
| `loading_60_80` | int | Stops with loading between 60% and 80%. |
| `loading_80_100` | int | Stops with loading over 80%. |

Loading data is only published by some train operators and is sparse — many services will have NULL `avg_loading`.

### The stops array

| Column | Type | Description |
|--------|------|-------------|
| `stops` | `array<struct<...>>` | One entry per calling point, ordered by scheduled time. |

Each element in the array is a struct with:

| Field | Type | Description |
|-------|------|-------------|
| `tpl` | string | TIPLOC code of the station. |
| `arr_sched` | string | Scheduled arrival time (`HH:MM:SS`). NULL at the origin. |
| `dep_sched` | string | Scheduled departure time (`HH:MM:SS`). NULL at the destination. |
| `arr_act` | string | Actual arrival time. NULL if the train did not reach this stop. |
| `dep_act` | string | Actual departure time. NULL if the train did not depart this stop. |
| `arr_delay` | int | Arrival delay in minutes (midnight-corrected). NULL if no actual arrival. |
| `dep_delay` | int | Departure delay in minutes (midnight-corrected). NULL if no actual departure. |
| `cancelled` | boolean | `true` if this stop was cancelled. |

To query the stops array in Athena, use `CROSS JOIN UNNEST`:

```sql
SELECT n.rid, stop.tpl, stop.arr_delay
FROM normalised_v1 n
CROSS JOIN UNNEST(n.stops) AS t(stop)
WHERE n.year = '2025' AND n.month = '10' AND n.day = '01'
  AND stop.tpl = 'PADTON'
```

Only locations where the train was scheduled to stop are included — pass-through timing points are excluded.

### Partition keys

| Column | Type | Description |
|--------|------|-------------|
| `year` | string | Service year (e.g., `'2025'`). |
| `month` | string | Service month (e.g., `'10'`). |
| `day` | string | Service day (e.g., `'01'`). |

**Always filter on partition keys.** Without them, Athena will scan the entire dataset — this is slow and expensive.

## Resolving station names

TIPLOC codes like `BRSTLTM` or `PADTON` are not particularly intuitive. The `tiplocs` reference table maps them to human-readable station names, three-letter CRS codes, and coordinates.

You can create the tiplocs table in Athena from the same S3 bucket, or simply use it as a lookup in your queries:

```sql
SELECT
  n.rid,
  n.toc,
  orig.StationName AS origin,
  dest.StationName AS destination,
  n.cancellation_status
FROM normalised_v1 n
LEFT JOIN tiplocs orig ON n.origin_tpl = orig.TiplocCode
LEFT JOIN tiplocs dest ON n.destination_tpl = dest.TiplocCode
WHERE n.year = '2025' AND n.month = '10' AND n.day = '01'
  AND n.passenger = true
LIMIT 20
```

One thing to watch: **29 stations have multiple TIPLOCs** sharing a single CRS code. Clapham Junction, for example, spans five TIPLOCs. If you are aggregating by station, group by `CrsCode` rather than `TiplocCode` and use `COUNT(DISTINCT rid)` to avoid double-counting services.

## Example queries

### How many passenger services ran on a given day?

```sql
SELECT
  COUNT(*) AS total_services,
  COUNT(*) FILTER (WHERE cancellation_status = 'ran') AS ran,
  COUNT(*) FILTER (WHERE cancellation_status = 'cancelled') AS cancelled,
  COUNT(*) FILTER (WHERE cancellation_status = 'partially_cancelled') AS partial
FROM normalised_v1
WHERE year = '2025' AND month = '10' AND day = '01'
  AND passenger = true
```

### Which operators had the highest cancellation rate last month?

```sql
SELECT
  toc,
  COUNT(*) AS total,
  COUNT(*) FILTER (WHERE cancellation_status IN ('cancelled', 'partially_cancelled')) AS cancellations,
  ROUND(
    100.0 * COUNT(*) FILTER (WHERE cancellation_status IN ('cancelled', 'partially_cancelled')) / COUNT(*),
    1
  ) AS cancel_pct
FROM normalised_v1
WHERE year = '2025' AND month = '09'
  AND passenger = true
GROUP BY toc
HAVING COUNT(*) > 100
ORDER BY cancel_pct DESC
```

### What is the average delay for trains calling at Bristol Temple Meads?

```sql
SELECT
  ROUND(AVG(stop.arr_delay), 1) AS avg_arr_delay_mins,
  COUNT(DISTINCT n.rid) AS num_services
FROM normalised_v1 n
CROSS JOIN UNNEST(n.stops) AS t(stop)
WHERE n.year = '2025' AND n.month = '10' AND n.day = '01'
  AND n.passenger = true
  AND stop.tpl = 'BRSTLTM'
  AND stop.arr_delay IS NOT NULL
```

### The most delayed services on a given day

```sql
SELECT
  n.rid,
  n.toc,
  orig.StationName AS origin,
  dest.StationName AS destination,
  n.origin_sched_dep,
  n.max_delay_mins,
  n.destination_delay_mins,
  n.delay_reason
FROM normalised_v1 n
LEFT JOIN tiplocs orig ON n.origin_tpl = orig.TiplocCode
LEFT JOIN tiplocs dest ON n.destination_tpl = dest.TiplocCode
WHERE n.year = '2025' AND n.month = '10' AND n.day = '01'
  AND n.passenger = true
  AND n.cancellation_status = 'ran'
ORDER BY n.max_delay_mins DESC
LIMIT 20
```

## Caveats

A few things to keep in mind when working with this data:

- **Always filter by partition keys.** Queries without `year`, `month`, and `day` filters will scan the entire dataset — this is both slow and expensive on Athena.
- **Data gaps exist.** There are 14 missing days in January 2025 and 10 in February 2025 due to ingestion outages. Plan around these if computing aggregates over those months.
- **Loading data is sparse.** Only some train operators publish passenger loading figures. Many rows will have NULL `avg_loading` — this does not mean the train was empty.
- **Multi-TIPLOC stations.** 29 stations have multiple TIPLOC codes sharing one CRS code. If aggregating by station, group by CRS code via the `tiplocs` table.
- **Requester Pays.** The S3 bucket uses Requester Pays pricing — you pay for data transfer costs when querying.
- **Non-passenger services are included.** The dataset contains empty stock movements and other non-passenger workings. Filter on `passenger = true` for passenger-only analysis.
- **cancel_reason coverage.** Cancellation reason text is not available for all cancelled services, particularly in the earlier months of 2025.

## What can you build?

This dataset powers the analysis on this blog — from [station cancellation rankings](/posts/uk-cancellations-2026/) to [estimating passengers affected by cancelled trains](/posts/passengers-affected-by-cancellations-2025/). A few ideas for what you could build:

- **Regional performance dashboards** — compare on-time performance across different parts of the network
- **Operator scorecards** — track cancellation and delay rates by train operating company over time
- **Commuter reliability indices** — measure how reliably specific routes run during peak hours using the stops array
- **Loading patterns** — identify overcrowded services and correlate with delays
- **Cancellation lead time analysis** — use `minutes_before_origin_dep` to study how far in advance cancellations are announced

The source code for the ingestion pipeline and normalisation process is available on [GitHub](https://github.com/iiAnderson/darwin-storage). If you build something with this data, I would love to hear about it — reach out at [rail@robbiea.co.uk](mailto:rail@robbiea.co.uk).
