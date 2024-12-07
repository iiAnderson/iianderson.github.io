---
layout: post
title:  "First new steps - Analysing the UK's busiest stations"
categories: [Railways, Analysis]
tags: [railway, analysis, sql, darwin]
---

As mentioned in my previous article, over the last few months I've been collecting data from National Rail's Darwin push port -  a comprehensive dataset that tracks all train movements across the UK. In this series of articles, I plan to delve deeper into the dataset, uncovering insights that will interest train enthusiasts, commuters, and government bodies alike.

This first article I want to start with a simple bit of analysis - What's the busiest station in the UK by number of services - but one that I was surprised by. Before we get to that however, let's explain what the data we'll be looking at is.

In the National Rail system, the definition of a "rail service" can be quite nuanced. Services may be split, rejoined, consist of empty stock movements (ECS), or even be freight trains. For this analysis, let’s focus on passenger services - excluding ECS and freight - which typically have an origin, a destination, and several intermediate locations along the way. These locations may include stations, but can also represent other points of importance on the railway, such as junctions. Each location is identified by a `tpl` (TIPLOC) code, while each service is assigned a unique `rid`.

For our first analysis, we will look at the number of unique passenger services that pass through each station in the UK, sorting from the busiest to the least busy. This will provide us a table of the number of `rid`s passing through a given `tpl`.
And here’s what the data looks like — for the 22nd of November.

| tpl     | Location Name                                     | count |
| ------- | ------------------------------------------------- | ----- |
| LNDNBDE | London Bridge (Eastern) (South Eastern Platforms) | 1302  |
| LEEDS   | Leeds                                             | 1237  |
| WATRLMN | Waterloo Main Lines                               | 1212  |
| WATRLWC | Waterloo West Crossings                           | 1212  |
| STFD    | Stratford (London)                                | 1160  |
| LIVST   | London Liverpool Street                           | 1018  |
| BTHNLGR | Bethnal Green                                     | 1015  |
| MNCRPIC | Manchester Piccadilly                             | 1006  |
| ARDWCKJ | Ardwick Junction                                  | 978   |
| WDON    | Wimbledon                                         | 944   |
| ECROYDN | East Croydon                                      | 943   |
| RAYNSPK | Raynes Park                                       | 943   |
| WNDMLBJ | Windmill Bridge Junction                          | 940   |
| CLPHMJM | Clapham Junction Main Lines                       | 935   |

On initial inspection, this dataset looks nothing like the 2024 figures for busiest station in the UK - I mean Bethnal Green is on there! So what's going on?

Well, as we said above - the key aspect is this is the number of unique services passing a given station - not necessarily stopping. Bethnal Green sits on the Weaver line of the London Overground, but also has the lines to London Liverpool St passing through it's bounds. This means services going to Liverpool St will report they've passed Bethnal Green. The same can be said for the non-station locations, Waterloo West Crossings (north of Clapham Junction), Ardwick Junction (south of Manchester Picadilly) and Windmill Bridge Junction (north of East Croydon) - each of these is a major junction which has many services passing through it.

There are a few odditites in this data though. Clapham Junction sits at the entrypoint for almost all services into Waterloo and Victoria - yet Waterloo is near the top and Clapham Junction is down in 14th. Why? The clue is in the name of the TPL Clapham Junction "Main Lines". This `tpl` covers just on of the routes into the station, and we can see this if we search the dataset for `tpl`'s start with CLPHM. We find:

| tpl     | Location Name                  | count |
| ------- | ------------------------------ | ----- |
| CLPHMJM | Clapham Junction Main Lines    | 935   |
| CLPHMJC | Clapham Junction Central Lines | 774   |
| CLPHMJW | Clapham Junction Windsor Lines | 351   |
| CLPHMJ1 | Clapham Junction Platform 1    | 302   |

This makes a lot more sense! We now get 2362 unique services passing through Clapham Junction - which is a far more plausible number than what we had previously. There is likely many other stations which require similar aggregation - and something I will be investigating in future posts.

If you want to re-create this analysis, you soon will be able to! I intend to post my dataset to the AWS Open Data Registry, where you will be able to setup AWS infrastructure to query the dataset yourself. I will also post all queries used in these analysis, so hopefully you can improve on my work.

##  Appendix: SQL Query

```
WITH passenger_services AS (
	SELECT rid,
		update_id,
		year,
		month,
		day
	from services
	where services.passenger = true
),
data AS (
	SELECT rid,
		tpl
	FROM passenger_services
		INNER JOIN locations ON passenger_services.update_id = locations.service_update_id
	WHERE passenger_services.year = '2024'
		AND passenger_services.month = '11'
		AND (
			passenger_services.day = '21'
			OR passenger_services.day = '22'
		)
		AND starts_with(passenger_services.rid, '20241122')
	GROUP BY rid,
		tpl
)
SELECT
	tpl,
	count(*),
	array_agg(rid)
from data
GROUP BY tpl
ORDER BY count(*) DESC
```