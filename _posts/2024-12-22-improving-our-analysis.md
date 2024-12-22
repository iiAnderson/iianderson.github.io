## Improving Our Analysis of Unique Rail Services

In our last article, we looked at how TPLs (locations on the rail network) don’t always correspond directly to stations. We discovered that Clapham Junction actually has 4 TPLs associated with different parts of its station, which affected our initial analysis of the number of unique rail services from each station.

Today, we’re going to apply this insight to the rest of our data, in order to get a more accurate count of unique services per station.

### The Challenge: How can we aggregate?

One approach could be to manually look up each TPL and find its corresponding location on the network. This could be done using the [Railway Codes website](http://www.railwaycodes.org.uk/crs/crss.shtm), and then aggregate them manually by location name. However, this would be incredibly time-consuming—there are potentially hundreds of aggregations needed across our dataset of 3000 TPLs. We need to find a more efficient solution.

### A New Dataset: National Rail's Station Reference Data

After scouring the web, I found a new dataset from the National Rail Marketplace titled the “Station Reference Data.” This dataset contains metadata for each station in the UK, including station names, TPLs, and two important identifiers: NLC and CRS. Yet more acronyms! As always, [Railway Codes](http://www.railwaycodes.org.uk/crs/crs0.shtm) helps clarify:

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
