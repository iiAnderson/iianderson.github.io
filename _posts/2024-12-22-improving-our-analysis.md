## Improving Our Analysis of Unique Rail Services

In our last article, we looked at how TPLs (locations on the rail network) don’t always correspond directly to stations. We discovered that Clapham Junction actually has 4 TPLs associated with different parts of its station, which affected our initial analysis of the number of unique rail services from each station.

Today, we’re going to apply this insight to the rest of our data, in order to get a more accurate count of unique services per station.

### The Challenge: How can we aggregate?

One approach could be to manually look up each TPL and find its corresponding location on the network. This could be done using the [Railway Codes website](http://www.railwaycodes.org.uk/crs/crss.shtm), and then aggregate them manually by location name. However, this would be incredibly time-consuming—there are potentially hundreds of aggregations needed across our dataset of 3000 TPLs. We need to find a more efficient solution.

### A New Dataset: National Rail's Station Reference Data

After scouring the web, I found a new dataset from the National Rail Marketplace titled the [Station Reference Data](https://raildata.org.uk/dataProduct/P-027a0f6c-b6dc-46eb-b743-5c6e73137aaf) This dataset contains metadata for each station in the UK, including station names, TPLs, and two important identifiers: NLC and CRS. Yet more acronyms! As always, [Railway Codes](http://www.railwaycodes.org.uk/crs/crs0.shtm) helps clarify:

- **NLC**: National Location Codes
- **CRS**: Customer Reservation System

Both of these identifiers are useful for our analysis. They map TPLs for the same station to a single identifier and generally don't include entries for non-stations. For our purposes, I chose to use the CRS identifier, as it felt more aligned with what I needed.

### Joining Data for Aggregation

Armed with these new identifiers, we can join them into our dataset to create a more accurate aggregated dataset of unique services per CRS. I also added aggregation by TOC (Train Operating Company) and filtered the dataset to only include trains that actually stop at the station.

### The Results: Aggregated Unique Services per CRS

Here’s the result of our aggregation:

| **CRS** | **Station Name**        | **total** | **South Western** | **Northern** | **London Overground** | **TfWales** | **C2C** | **Chiltern** | **Cross Country** | **East Midlands** | **Gatwick Express** | **Great Northern** | **Thameslink** | **Grand Central** | **Great Western** | **Greater Anglia** | **Heathrow** | **West Midlands** | **London Transport** | **Merseyrail** | **Tyne & Wear** | **ScotRail** | **SouthEastern** | **Southern** | **Elizabeth Line** | **Avanti** | **LNER** |
| ------- | ----------------------- | --------- | ----------------- | ------------ | --------------------- | ----------- | ------- | ------------ | ----------------- | ----------------- | ------------------- | ------------------ | -------------- | ----------------- | ----------------- | ------------------ | ------------ | ----------------- | -------------------- | -------------- | --------------- | ------------ | ---------------- | ------------ | ------------------ | ---------- | -------- |
| **CLJ** | Clapham Junction        | 2085      | 1077              | 0            | 307                   | 0           | 0       | 0            | 0                 | 0                 | 0                   | 0                  | 0              | 0                 | 0                 | 0                  | 0            | 0                 | 0                    | 0              | 0               | 0            | 0                | 701          | 0                  | 0          | 0        |
| **LBG** | London Bridge           | 1702      | 0                 | 0            | 0                     | 0           | 0       | 0            | 0                 | 0                 | 0                   | 0                  | 368            | 0                 | 0                 | 0                  | 0            | 0                 | 0                    | 0              | 0               | 0            | 937              | 397          | 0                  | 0          | 0        |
| **LDS** | Leeds                   | 1251      | 0                 | 818          | 0                     | 0           | 0       | 0            | 128               | 0                 | 0                   | 0                  | 0              | 0                 | 0                 | 0                  | 0            | 0                 | 0                    | 0              | 0               | 0            | 0                | 0            | 0                  | 0          | 69       |
| **WAT** | London Waterloo         | 1213      | 1213              | 0            | 0                     | 0           | 0       | 0            | 0                 | 0                 | 0                   | 0                  | 0              | 0                 | 0                 | 0                  | 0            | 0                 | 0                    | 0              | 0               | 0            | 0                | 0            | 0                  | 0          | 0        |
| **SRA** | Stratford (London)      | 1138      | 0                 | 0            | 315                   | 0           | 0       | 0            | 0                 | 0                 | 0                   | 0                  | 0              | 0                 | 0                 | 471                | 0            | 0                 | 0                    | 0              | 0               | 0            | 0                | 0            | 352                | 0          | 0        |
| **VIC** | London Victoria         | 1120      | 0                 | 0            | 0                     | 0           | 0       | 0            | 0                 | 0                 | 72                  | 0                  | 1              | 0                 | 0                 | 0                  | 0            | 0                 | 0                    | 0              | 0               | 0            | 383              | 664          | 0                  | 0          | 0        |
| **LST** | London Liverpool Street | 1030      | 0                 | 0            | 321                   | 0           | 0       | 0            | 0                 | 0                 | 0                   | 0                  | 0              | 0                 | 0                 | 691                | 0            | 0                 | 0                    | 0              | 0               | 0            | 0                | 0            | 18                 | 0          | 0        |
| **MAN** | Manchester Piccadilly   | 1017      | 0                 | 603          | 0                     | 76          | 0       | 0            | 61                | 32                | 0                   | 0                  | 0              | 0                 | 0                 | 0                  | 0            | 0                 | 0                    | 0              | 0               | 0            | 0                | 0            | 0                  | 94         | 0        |
| **GLC** | Glasgow Central         | 920       | 0                 | 0            | 0                     | 0           | 0       | 0            | 3                 | 0                 | 0                   | 0                  | 0              | 0                 | 0                 | 0                  | 0            | 0                 | 0                    | 0              | 0               | 851          | 0                | 0            | 0                  | 40         | 2        |
| **ECR** | East Croydon            | 888       | 0                 | 0            | 0                     | 0           | 0       | 0            | 0                 | 0                 | 1                   | 0                  | 321            | 0                 | 0                 | 0                  | 0            | 0                 | 0                    | 0              | 0               | 0            | 0                | 566          | 0                  | 0          | 0        |
| **BHM** | Birmingham New Street   | 874       | 0                 | 0            | 0                     | 5           | 0       | 0            | 247               | 0                 | 0                   | 0                  | 0              | 0                 | 0                 | 0                  | 0            | 533               | 0                    | 0              | 0               | 0            | 0                | 0            | 0                  | 89         | 0        |
| **HYM** | Haymarket               | 843       | 0                 | 0            | 0                     | 0           | 0       | 0            | 5                 | 0                 | 0                   | 0                  | 0              | 0                 | 0                 | 0                  | 0            | 0                 | 0                    | 0              | 0               | 792          | 0                | 0            | 0                  | 15         | 15       |
| **EDB** | Edinburgh               | 822       | 0                 | 0            | 0                     | 0           | 0       | 0            | 30                | 0                 | 0                   | 0                  | 0              | 0                 | 0                 | 0                  | 0            | 0                 | 0                    | 0              | 0               | 667          | 0                | 0            | 0                  | 15         | 62       |
| **WIJ** | Willesden Junction      | 800       | 0                 | 0            | 477                   | 0           | 0       | 0            | 0                 | 0                 | 0                   | 0                  | 0              | 0                 | 0                 | 0                  | 0            | 0                 | 323                  | 0              | 0               | 0            | 0                | 0            | 0                  | 0          | 0        |


### Insights

It makes a lot more sense now! We can see that Clapham Junction tops the list, with an incredible number of unique services per day, followed closely by London Bridge. Then we have other major London commuter stations like Waterloo, Stratford, Victoria, and Liverpool Street, along with larger city hubs like Leeds, Manchester, and Glasgow — all of which have extensive metro services.

Interestingly, there’s one "rogue" entry: Willesden Junction. This station appears due to a quirk in the data. London Overground services run across Network Rail tracks, so they’re tracked in this dataset. Additionally, we also get some London Underground services where the Underground runs on National Rail tracks. This happens in a few locations:

- The Bakerloo line joins the Overground north of Queens Park and runs through Willesden Junction to Harrow and Wealdstone.
- The District line runs on Overground tracks to Richmond.
- The District line runs on Network Rail tracks between East Putney and Wimbledon.

### Conclusion

That’s it for today! As always, the Appendix will contain a link to the full dataset, along with the SQL queries used for this analysis.

## Appendix:

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
toc_for_rid AS (
	SELECT rid,
		toc
	FROM service_details
	WHERE toc <> ''
	GROUP BY rid,
		toc
),
data AS (
	SELECT passenger_services.rid,
		toc_for_rid.toc,
		tpl,
		passenger_services.day
	FROM passenger_services
		INNER JOIN locations ON passenger_services.update_id = locations.service_update_id
		INNER JOIN toc_for_rid ON toc_for_rid.rid = passenger_services.rid
	WHERE passenger_services.year = '2024'
		AND passenger_services.month = '11'
		AND (locations.type = 'ARR' OR locations.type = 'DEP')
	GROUP BY passenger_services.rid,
		toc,
		tpl,
		passenger_services.day
),
remapped AS (
    SELECT rid, day, station_data.crs, station_data.name, toc
    FROM data
    LEFT JOIN station_data ON data.tpl = station_data.tiploc
),
aggregated AS (
	SELECT crs,
	    name,
		day,
		toc,
		CASE
			WHEN toc = 'SW' THEN 1 ELSE 0
		END as SW,
		CASE
			WHEN toc = 'NT' THEN 1 ELSE 0
		END as NT,
		CASE
			WHEN toc = 'LO' THEN 1 ELSE 0
		END as LO,
		CASE
			WHEN toc = 'AW' THEN 1 ELSE 0
		END as AW,
		CASE
			WHEN toc = 'CC' THEN 1 ELSE 0
		END as CC,
		CASE
			WHEN toc = 'CH' THEN 1 ELSE 0
		END as CH,
		CASE
			WHEN toc = 'XC' THEN 1 ELSE 0
		END as XC,
		CASE
			WHEN toc = 'EM' THEN 1 ELSE 0
		END as EM,
		CASE
			WHEN toc = 'HT' THEN 1 ELSE 0
		END as HT,
		CASE
			WHEN toc = 'GX' THEN 1 ELSE 0
		END as GX,
		CASE
			WHEN toc = 'GN' THEN 1 ELSE 0
		END as GN,
		CASE
			WHEN toc = 'TL' THEN 1 ELSE 0
		END as TL,
		CASE
			WHEN toc = 'GC' THEN 1 ELSE 0
		END as GC,
		CASE
			WHEN toc = 'GW' THEN 1 ELSE 0
		END as GW,
		CASE
			WHEN toc = 'LE' THEN 1 ELSE 0
		END as LE,
		CASE
			WHEN toc = 'HC' THEN 1 ELSE 0
		END as HC,
		CASE
			WHEN toc = 'HX' THEN 1 ELSE 0
		END as HX,
		CASE
			WHEN toc = 'LM' THEN 1 ELSE 0
		END as LM,
		CASE
			WHEN toc = 'LT' THEN 1 ELSE 0
		END as LT,
		CASE
			WHEN toc = 'ME' THEN 1 ELSE 0
		END as ME,
		CASE
			WHEN toc = 'TW' THEN 1 ELSE 0
		END as TW,
		CASE
			WHEN toc = 'SR' THEN 1 ELSE 0
		END as SR,
		CASE
			WHEN toc = 'SE' THEN 1 ELSE 0
		END as SE,
		CASE
			WHEN toc = 'SN' THEN 1 ELSE 0
		END as SN,
		CASE
			WHEN toc = 'XR' THEN 1 ELSE 0
		END as XR,
		CASE
			WHEN toc = 'TP' THEN 1 ELSE 0
		END as TP,
		CASE
			WHEN toc = 'VT' THEN 1 ELSE 0
		END as VT,
		CASE
			WHEN toc = 'GR' THEN 1 ELSE 0
		END as GR
	from remapped
	WHERE day = '19'
)
SELECT crs,
    name,
	day,
	count(*) as total,
	sum(SW) as "South Western",
	sum(NT) as "Northern",
	sum(LO) as "London Overground",
	sum(AW) as "TfWales",
	sum(CC) as "C2C",
	sum(CH) as "Chiltern",
	sum(XC) as "Cross Country",
	sum(EM) as "East Midlands",
	sum(HT) as "Hull Trains",
	sum(GX) as "Gatwick Express",
	sum(GN) as "Great Northern",
	sum(TL) as "Thameslink",
	sum(GC) as "Grand Central",
	sum(GW) as "Great Western",
	sum(LE) as "Greater Anglia",
	sum(HC) + sum(HX) as "Heathrow",
	sum(LM) as "West Midlands",
	sum(LT) as "London Transport",
	sum(ME) as "Merseyrail",
	sum(TW) as "Tyne & Wear",
	sum(SR) as "ScotRail",
	sum(SE) as "SouthEastern",
	sum(SN) as "Southern",
	sum(XR) as "Elizabeth Line",
	sum(VT) as "Avanti",
	sum(GR) as "LNER"
	from aggregated
WHERE DAY = '19' AND name <> ''
GROUP BY crs, name,
	day
ORDER BY count(*) DESC

```