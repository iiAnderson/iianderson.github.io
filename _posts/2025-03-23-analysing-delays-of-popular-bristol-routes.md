# Analysing popular routes out of Bristol Temple Meads

Over the past few months, I have been designing a system for ingesting data from National Rail’s “Darwin Push Port Feed”. This feed contains live information about train movements, scheduling, and service alterations across the UK rail network. In this article, I will share some analysis of the service delays for three most popular (by passenger numbers) routes out of Bristol Temple Meads. These are the route to London Paddington, Cardiff Central and Clifton Down.

## The Query

To obtain this data, I used the query found in the appendix, it does a few things of note:
- It fetches the last SCHED and ACT (scheduled and actual) messages for each tpl. In this case, it's assumed the last schedule message is the most accurate.
- It attempts to correct for day rollover (where a service runs into the next day) when computing the delay for a service. This is done by comparing the schedule and actual arrival times, and if the difference is off by more than half a day (I assume no service is this late!) then it adjusts the calculation to ensure the correct delay is calculated.
- It assumees that the last and first locations updates are the destination and origin repsectively. This could be incorrect if there is data missing (ie I fail to capture the start of a service).

This query was then run for each three routes, pulling back the raw data into Google Sheets for analysis. 

## The Analysis

To analyse the data, I needed to ensure only the correct data was being pulled in, as my query cannot currently understand directionality of a route. For example, a service can be arriving in Bristol Temple Meads from either Weston (en route to Paddington) or from Paddington (either terminating or continuing to Weston). This was done by filtering the data for only services that started in the correct direction - in this example, if I want to analyse services going to London Paddington via Bristol, they would need to terminate at either Taunton, Weston or the South West of Wales.

Now that's done, we can see some lovely heat maps!

### Services to London Paddington

![Alt text](https://blog.robbiea.co.uk/assets/img/BRI-PAD-March25.png "Bristol to Paddington delays by hour across march")

### Services to Cardiff Central

![Alt text](https://blog.robbiea.co.uk/assets/img/BRI-CDF-March25.png "Bristol to Cardiff delays by hour across march")

### Services to Clifton Down

![Alt text](https://blog.robbiea.co.uk/assets/img/BRI-CFN-March25.png "Bristol to Clifton Down delays by hour across march")

Instantly these charts show an interesting picture, each route has approximatley the same number of services per hour (approx 2), but we can clearly see the Clifton Down route have far fewer delays than the other routes - although this could be due to the reduced distance it covers.

The Paddington services show a different story, only 5 of the days tested have a maximum delayed of less than 15 minutes! There seems to be little pattern to the delays across a given day - although the last service of the day seems especially prone to delays.

The Cardiff services show a similar amount of delays - although there are far fewer services which are not delayed at all. This however is the first route which has two different sections - Bristol to Cardiff services can come either from Taunton or Portsmouth. We can generate heatmaps for these separatley:

Services via Taunton
![Alt text](https://blog.robbiea.co.uk/assets/img/BRI-CDF-TAU-March25.png "Bristol to Cardiff delays on the Taunton route by hour across march")

Services via Portsmouth
![Alt text](https://blog.robbiea.co.uk/assets/img/BRI-CDF-PHBR-March25.png "Bristol to Cardiff delays on the Portsmouth route by hour across march")

These extra heatmaps give us a little more insight into the source of the delays. I initally suspected the Portsmouth route due to the congestion around the south coast, but it seems the route towards Taunton illicits more delays on average.


This is just a first pass analysis. In future posts, I'd like to look further at where these delays originate - and if it can show us bottlenecks in the network. 

## Appendix

```
WITH data AS (
	SELECT services_v1.rid,
		ts,
		tpl,
		type,
		time_type,
		time
	FROM services_v1
		INNER JOIN locations_v1 ON services_v1.update_id = locations_v1.service_update_id
		INNER JOIN service_details ON services_v1.rid = service_details.rid
	WHERE service_details.passenger = true
		AND (services_v1.year = '2025')
		AND (services_v1.month = '03')
		AND starts_with(services_v1.rid, '202503')
		AND (locations_v1.year = '2025')
		AND (locations_v1.month = '03')
),
sched_int AS (
	SELECT rid,
		tpl,
		ts,
		type,
		time_type,
		time,
		ROW_NUMBER() OVER (
			PARTITION BY rid,
			tpl,
			type
			ORDER BY ts DESC
		) AS row_number
	from data
	WHERE time_type = 'SCHED'
),
sched AS (
	SELECT *
	FROM sched_int
	WHERE row_number = 1
	ORDER BY time ASC
),
act_int AS (
	SELECT rid,
		tpl,
		ts,
		type,
		time_type,
		time,
		ROW_NUMBER() OVER (
			PARTITION BY rid,
			tpl,
			type
			ORDER BY ts DESC
		) AS row_number
	from data
	WHERE time_type = 'ACT'
),
act AS (
	SELECT *
	FROM act_int
	WHERE row_number = 1
	ORDER BY time ASC
),
service_delays AS (
	SELECT act.rid,
		act.tpl,
		act.ts,
		sched.time AS sched_time,
		act.time AS act_time,
		CASE
			when DATE_DIFF(
				'minute',
				date_parse(sched.time, '%H:%i:%S'),
				date_parse(act.time, '%H:%i:%S')
			) < -800 then DATE_DIFF(
				'minute',
				date_parse(sched.time, '%H:%i:%S'),
				date_parse(act.time, '%H:%i:%S')
			) + 1440
			when DATE_DIFF(
				'minute',
				date_parse(sched.time, '%H:%i:%S'),
				date_parse(act.time, '%H:%i:%S')
			) > 900 then DATE_DIFF(
				'minute',
				date_parse(sched.time, '%H:%i:%S'),
				date_parse(act.time, '%H:%i:%S')
			) - 1440 else DATE_DIFF(
				'minute',
				date_parse(sched.time, '%H:%i:%S'),
				date_parse(act.time, '%H:%i:%S')
			)
		END as delay,
		sched.type
	FROM act
		LEFT JOIN sched ON act.tpl = sched.tpl
		AND act.rid = sched.rid
		AND act.type = sched.type
),
all_services AS (
	SELECT rid,
		array_agg(tpl) as tpls
	FROM data
	GROUP BY rid
),
filtered_services AS (
	SELECT rid,
		tpls
	FROM all_services
	WHERE contains(tpls, 'BRSTLTM')
		AND contains(tpls, 'CRDFCEN')
),
destination_int AS (
	SELECT ROW_NUMBER() OVER (
			PARTITION BY rid
			ORDER BY act_time ASC
		) AS destination,
		rid,
		tpl
	FROM service_delays
),
destinations AS (
	SELECT *
	FROM destination_int
	WHERE destination = 1
),
origin_int AS (
	SELECT ROW_NUMBER() OVER (
			PARTITION BY rid
			ORDER BY act_time DESC
		) AS origin,
		rid,
		tpl
	FROM service_delays
),
origins AS (
	SELECT *
	FROM origin_int
	WHERE origin = 1
)
SELECT filtered_services.rid,
	SUBSTR(filtered_services.rid, 1, 8) as date,
	service_delays.tpl,
	ts,
	sched_time,
	act_time,
	delay,
	type,
	origins.tpl,
	destinations.tpl,
	ROW_NUMBER() OVER (
		PARTITION BY SUBSTR(filtered_services.rid, 1, 8)
		ORDER BY act_time DESC
	) as s_id
FROM filtered_services
	LEFT JOIN service_delays ON service_delays.rid = filtered_services.rid
	LEFT JOIN origins ON service_delays.rid = origins.rid
	LEFT JOIN destinations ON service_delays.rid = destinations.rid
WHERE starts_with(filtered_services.rid, '202503')
	AND service_delays.tpl = 'CRDFCEN'
	AND type = 'ARR'
ORDER BY rid,
	sched_time
```