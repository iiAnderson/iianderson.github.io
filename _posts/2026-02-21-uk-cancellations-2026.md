---
layout: post
title: "The UK train station with most cancellations in 2026"
categories: [Railways, Analysis]
tags: [railway, analysis, visualization, darwin, cancellations, uk]
---

# Train cancellations 2026

Over the past year, I have been collecting train service data from National Rail - storing each train's movements, delays and cancellations. I am now working on developing new analysis and visualisations to show this data, along with holding the bodies that run our railways accountable. In today's article, let's explore cancellations across the network in 2026 - where are they happening and why.

## Dataset

Before we start on that though, I'd like to discuss the dataset we're operating on. The raw data collected from National Rail isn't very helpful - it's a new message each time a train reaches a station, along with updates on that service's estimated later stops. This is a lot of data - for 2026 alone it's above 500 million rows. Analysing a dataset this size is starting to stretch into expensive territory (you'll notice all my previous analysis focuses on a single location - this is for a reason!), so I wanted to find a way to reduce this cost.

To the rescue is a common data engineering tactic: normalisation. Taking our complex, often messy, rows of train locations and marshalling them into a single, standardised format for each service that we can quickly operate over. Doing this, I reduced down our dataset to ~1.5 million rows - much more manageable!

## Query

Let's build a query! There are a few things we need to consider:

- Train locations are identified by TIPLOC codes. Large stations may have multiple codes however. We can use a different identifier, the CRS Code, to do this.
- We want to count distinct services cancelled. If a service visits the same station twice (for example, it's a loop), we only want this counted once.
- We don't want to include stations where a train passes - just passenger stops.
- We want to filter out non-passenger services, such as empty stock movements.

With that, we have our query! You can see it in the appendix below. Now, onto the results.

## Results

Here are the first thirty results, ordered by cancellation rate (percent of services cancelled).

| # | Station ID | Station Name | Total Services | Total Cancelled | Full Cancellations | Partial Cancellations | Cancellation Rate (%) |
|---|-----------|--------------|---------------|----------------|-------------------|----------------------|----------------------|
| 1 | SKN | St Keyne Wishing Well Halt | 149 | 56 | 56 | 0 | 37.6 |
| 2 | CAU | Causeland | 149 | 56 | 56 | 0 | 37.6 |
| 3 | KGN | Kings Nympton | 507 | 112 | 42 | 70 | 22.1 |
| 4 | MRD | Morchard Road | 1,231 | 262 | 91 | 171 | 21.3 |
| 5 | COP | Copplestone | 1,251 | 265 | 94 | 171 | 21.2 |
| 6 | YEO | Yeoford | 1,266 | 269 | 98 | 171 | 21.2 |
| 7 | UMB | Umberleigh | 1,274 | 268 | 97 | 171 | 21.0 |
| 8 | EGG | Eggesford | 1,305 | 269 | 99 | 170 | 20.6 |
| 9 | OLY | Ockley | 1,229 | 250 | 31 | 219 | 20.3 |
| 10 | HLM | Holmwood | 1,237 | 226 | 7 | 219 | 18.3 |
| 11 | LAP | Lapford | 551 | 101 | 29 | 72 | 18.3 |
| 12 | WNH | Warnham | 1,237 | 226 | 7 | 219 | 18.3 |
| 13 | NQY | Newquay | 648 | 112 | 104 | 8 | 17.3 |
| 14 | LUX | Luxulyan | 599 | 103 | 93 | 10 | 17.2 |
| 15 | CPN | Chapelton (Devon) | 217 | 37 | 16 | 21 | 17.1 |
| 16 | BGL | Bugle | 624 | 106 | 96 | 10 | 17.0 |
| 17 | PMA | Portsmouth Arms | 255 | 43 | 20 | 23 | 16.9 |
| 18 | ROC | Roche | 624 | 105 | 96 | 9 | 16.8 |
| 19 | QUI | Quintrell Downs | 624 | 104 | 96 | 8 | 16.7 |
| 20 | SCR | St Columb Road | 624 | 104 | 96 | 8 | 16.7 |
| 21 | THS | Thurso | 310 | 51 | 21 | 30 | 16.5 |
| 22 | GGJ | Georgemas Junction | 310 | 47 | 21 | 26 | 15.2 |
| 23 | SDP | Sandplace | 584 | 78 | 78 | 0 | 13.4 |
| 24 | LOO | Looe | 720 | 95 | 95 | 0 | 13.2 |
| 25 | KBC | Kinbrace | 273 | 34 | 20 | 14 | 12.5 |
| 26 | SCT | Scotscalder | 273 | 34 | 20 | 14 | 12.5 |
| 27 | ABC | Altnabreac | 273 | 34 | 20 | 14 | 12.5 |
| 28 | KIL | Kildonan | 310 | 38 | 21 | 17 | 12.3 |
| 29 | FRS | Forsinard | 310 | 38 | 21 | 17 | 12.3 |
| 30 | WCK | Wick | 310 | 37 | 21 | 16 | 11.9 |

Unless you're a real rail nut (or a local) I imagine you've not heard of quite a few of these stations. That's because they cluster around three branch lines:

- **Looe Valley line** (Cornwall, to Looe)
- **Atlantic Coast line** (Cornwall, to Newquay)
- **Far North line** (Scotland, to Thurso and Wick)

And one mainline, the **Mole Valley Line** linking Horsham and Epsom.

This makes sense, as the monumental rainfall in Jan & Feb has closed both the Looe Valley and Atlantic Coast lines to all train traffic for significant periods of time. As far as I can tell, the Far North line hasn't been closed, but considering its remoteness it's not too much of a surprise it features in this list.

We can see these stations visualised below:

<iframe src="/assets/visualizations/top-cancellations-uk-2026.html" width="100%" height="950" frameborder="0" style="border: 1px solid #ddd; border-radius: 8px; margin: 20px 0;"></iframe>

## Results: Part 2

Now we can run more efficient queries, why limit ourselves to 30 stations! Let's generate a visualisation across the entire UK.

*[Visualisation coming soon]*
