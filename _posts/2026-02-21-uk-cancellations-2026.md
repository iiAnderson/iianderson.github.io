---
layout: post
title: "The UK train station with most cancellations in 2026"
categories: [Railways, Analysis]
tags: [railway, analysis, visualization, darwin, cancellations, uk]
---

# Train cancellations 2026

Over the past year, I have been collecting train service data from National Rail - storing each train's movements, delays and cancellations. I am now working on developing new analysis and visualisations to show this data, along with holding the bodies that run our railways accountable. In today's article, let's explore cancellations across the network in 2026 - where are they happening and why.

## The dataset

Before we get into the results , I'd like to touch on the dataset itself. The raw data from National Rail is huge - every time a train reaches a station, a new message is generated, along with updates on estimated times for later stops. For 2026 alone, that's over **500 million rows**. Analysing data at this scale can get expensive fast (you may have noticed that all my previous analysis focuses on a single location - that's not a coincidence!).

To bring the cost down, I used a common data engineering technique: **normalisation**. Taking our complex, often messy, rows of train locations and marshalling them into a single, standardised format for each service that we can quickly operate over. Doing this, I reduced down our dataset to ~1.5 million rows - much more manageable!

## Building the query

Let's build a query! There are a few things we need to consider:

- Train locations are identified by **TIPLOC codes** but large stations can have several of these. By looking up the **CRS Code** from the TIPLOC we can aggregate these results into a single statino.
- We want to count **distinct services cancelled**. If a service visits the same station twice (for example, it's a loop), we only want this counted once.
- We're only interested in **passenger stops** — not stations a train passes through without stopping.
- We want to filter out **non-passenger services**, such as empty stock movements.

With that, we have our query! You can see it in the appendix below. Now, onto the results.

## Results

Here are the top thirty results, ordered by cancellation rate - the percentage of scheduled services that were cancelled.

| # | Station | Total Services | Cancelled | Full | Partial | Rate |
|---|---------|---------------|-----------|------|---------|------|
| 1 | St Keyne Wishing Well Halt | 149 | 56 | 56 | 0 | 37.6% |
| 2 | Causeland | 149 | 56 | 56 | 0 | 37.6% |
| 3 | Kings Nympton | 507 | 112 | 42 | 70 | 22.1% |
| 4 | Morchard Road | 1,231 | 262 | 91 | 171 | 21.3% |
| 5 | Copplestone | 1,251 | 265 | 94 | 171 | 21.2% |
| 6 | Yeoford | 1,266 | 269 | 98 | 171 | 21.2% |
| 7 | Umberleigh | 1,274 | 268 | 97 | 171 | 21.0% |
| 8 | Eggesford | 1,305 | 269 | 99 | 170 | 20.6% |
| 9 | Ockley | 1,229 | 250 | 31 | 219 | 20.3% |
| 10 | Holmwood | 1,237 | 226 | 7 | 219 | 18.3% |
| 11 | Lapford | 551 | 101 | 29 | 72 | 18.3% |
| 12 | Warnham | 1,237 | 226 | 7 | 219 | 18.3% |
| 13 | Newquay | 648 | 112 | 104 | 8 | 17.3% |
| 14 | Luxulyan | 599 | 103 | 93 | 10 | 17.2% |
| 15 | Chapelton (Devon) | 217 | 37 | 16 | 21 | 17.1% |
| 16 | Bugle | 624 | 106 | 96 | 10 | 17.0% |
| 17 | Portsmouth Arms | 255 | 43 | 20 | 23 | 16.9% |
| 18 | Roche | 624 | 105 | 96 | 9 | 16.8% |
| 19 | Quintrell Downs | 624 | 104 | 96 | 8 | 16.7% |
| 20 | St Columb Road | 624 | 104 | 96 | 8 | 16.7% |
| 21 | Thurso | 310 | 51 | 21 | 30 | 16.5% |
| 22 | Georgemas Junction | 310 | 47 | 21 | 26 | 15.2% |
| 23 | Sandplace | 584 | 78 | 78 | 0 | 13.4% |
| 24 | Looe | 720 | 95 | 95 | 0 | 13.2% |
| 25 | Kinbrace | 273 | 34 | 20 | 14 | 12.5% |
| 26 | Scotscalder | 273 | 34 | 20 | 14 | 12.5% |
| 27 | Altnabreac | 273 | 34 | 20 | 14 | 12.5% |
| 28 | Kildonan | 310 | 38 | 21 | 17 | 12.3% |
| 29 | Forsinard | 310 | 38 | 21 | 17 | 12.3% |
| 30 | Wick | 310 | 37 | 21 | 16 | 11.9% |

Note: Partially cancelled services are classed by a train stopping at the marked station, but not completing it's entire route. 

Unless you're a real rail nut (or a local) I'd guess you've not heard of quite a few of these stations. That's because they cluster around five lines:
- **Tarka line** (Cornwall, Exeter to Barnstaple)
- **Looe Valley line** (Cornwall, to Looe)
- **Atlantic Coast line** (Cornwall, to Newquay)
- **Far North line** (Scotland, to Thurso and Wick)
- **Mole Valley line** (Sussex, to Horsham and Epsom)

The Cornish lines are perhaps the least surprising entry here. The monumental rainfall in January and February closed both the Looe Valley and Atlantic Coast lines to all traffic for significant stretches of time. This also applies to the Mole Valley line, which suffered a landslip at the end of January. 

The Far North line tells a different story: as far as I can tell it hasn't faced the same weather-related closures, but given how remote it is (Altnabreac, for instance, is arguably the most isolated station in Britain), it's not a huge shock to see it featurin

We can see these stations visualised below:

<iframe src="/assets/visualizations/top-cancellations-uk-2026.html" width="100%" height="650" frameborder="0" style="border: 1px solid #ddd; border-radius: 8px; margin: 20px 0;"></iframe>

## Results: A bigger picture

Now we can run more efficient queries, why limit ourselves to 30 stations! Let's generate a visualisation across the entire UK.

<iframe src="/assets/visualizations/all-cancellations-uk-2026.html" width="100%" height="950" frameborder="0" style="border: 1px solid #ddd; border-radius: 8px; margin: 20px 0;"></iframe>

From this, we can see our top 5 delayed routes clearly - orange in a sea of green! We can also see other lines which struggle with cancellations, such as the stunning Heart of Wales line. You'll also notice that switching to "Number Cancelled" shows a very different picture, with most lines having some level of cancelled services - with only the lines north of Ipswich excelling in minimising the services cancelled.

In future articles, I will look further back in cancellations, and see what patterns we can uncover.


## Appendix

SQL to compute the above datasets

```
-- Stations ranked by cancellation percentage for 2026 year-to-date.
-- Uses normalised_v1 (one row per service) with the stops array for efficient station filtering.
-- Pass-through timing points excluded via arr_sched/dep_sched filter.
-- Multi-TIPLOC stations (e.g. Clapham Junction) aggregated via CRS code.

WITH all_stops AS (
    SELECT
        stop.tpl           AS tpl,
        rid,
        cancellation_status,
        stop.cancelled     AS stop_cancelled
    FROM normalised_v1
    CROSS JOIN UNNEST(stops) AS t(stop)
    WHERE year = '2026'
      AND passenger = true
      AND (stop.arr_sched IS NOT NULL OR stop.dep_sched IS NOT NULL)
),

station_stats AS (
    SELECT
        COALESCE(t.CrsCode, s.tpl)                                              AS station_id,
        MIN(t.StationName)                                                       AS station_name,
        COUNT(DISTINCT s.rid)                                                    AS total_services,
        COUNT(DISTINCT CASE
            WHEN s.stop_cancelled = true
             AND s.cancellation_status IN ('cancelled', 'partially_cancelled')
            THEN s.rid END)                                                      AS total_cancelled_services,
        COUNT(DISTINCT CASE
            WHEN s.stop_cancelled = true
             AND s.cancellation_status = 'cancelled'
            THEN s.rid END)                                                      AS from_full_cancellations,
        COUNT(DISTINCT CASE
            WHEN s.stop_cancelled = true
             AND s.cancellation_status = 'partially_cancelled'
            THEN s.rid END)                                                      AS from_partial_cancellations
    FROM all_stops s
    LEFT JOIN tiplocs t ON s.tpl = t.TiplocCode
    GROUP BY COALESCE(t.CrsCode, s.tpl)
)

SELECT
    station_id,
    station_name,
    total_services,
    total_cancelled_services,
    from_full_cancellations,
    from_partial_cancellations,
    ROUND(100.0 * total_cancelled_services / total_services, 1) AS cancellation_rate_pct
FROM station_stats
WHERE total_services >= 100
ORDER BY cancellation_rate_pct DESC
LIMIT 30
```