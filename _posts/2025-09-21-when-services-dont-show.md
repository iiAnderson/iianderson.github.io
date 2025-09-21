---
layout: post
title:  "Mind the Data Gap: Part 3. When services don't show up"
categories: [Railways, Analysis]
tags: [railway, analysis, sql, darwin]
---

# When services don't show up

Over the last year, I have been designing a system for ingesting data from National Rail's "Darwin Push Port Feed". This feed contains live information about train movements, scheduling, and service alterations across the UK rail network. Having previously focused on delays and routing patterns, today we're diving into cancelled services - investigating how cancellations affect stations across the network.

The analysis covers September 11-19 2025, examining 22,782 station-day records across 2,548 stations. 

Previously, we've been ignoring cancellation data, opting instead to focus on scheduled, estimated and actual times services passed stations. To capture this data, we need to parse a new property on a location in the darwin data feed, `@can`. This is a boolean property (although only appears when set to true) that indicates if a service has been cancelled at the given TIPLOC. You can see an example of a passing location that has been cancelled below.

```json
{
    "@wtp": "10:26",
    "@tpl": "NEWMLDN",
    "@can": "true"
}
```

A TIPLOC can be cancelled for several reasons. A service might "skip" stations due to delays, marking those TIPLOCs as cancelled while continuing to its destination. Alternatively, a service can be outright cancelled, marking all future locations as cancelled. For our analysis, we count any cancellation affecting a TIPLOC - whether from skipping or complete service cancellation.

The data was pulled out of AWS Athena via SQL, whose output consisted of the day, the TIPLOC, the number of services that called at the station, the number of these that were cancelled, and a percentage of cancelled services. After a few days of collecting data, we were able to start seeing some patterns. 

## Top Performers

Across the analysis period, 487 stations (19.3% of the network) achieved perfect reliability with zero cancelled services. The standout performer was **Huddersfield**, handling 3,752 services across 9 days without a single cancellation - suggesting that high volume doesn't preclude perfect performance.

There was a mix of stations included here too, some commuter stations such as **Southend Victoria**, rural stations like **Llantwit Major**, newer airport links to **Heathrow Airport Terminal 5** and older metro systems such as stations on the **Tyne and Wear metro** lines. 

## Poor Performers

Unlike the randomly distributed top performers, poor performance shows clear geographic clustering. The Heart of Wales line dominates the worst performers, with 13 of the top 15 stations. This seems to be due to ongoing engineering works on the line.

Several other problematic corridors emerged:

- **Thameslink's Sutton Loop**: All stations feature in the top 25, with ~16% cancellation rates
- **Portsmouth Harbour**: ~15% cancellation rate - odd considering there are no other nearby stations in the list.
- **TfW Llandudno-Blaenau Ffestiniog branch**: ~13% cancellation rate
- **GWR Yeovil-Weymouth line**: Multiple stations in top 50, ~12% cancellation rate

Looking further into Portsmouth Harbour, it seems services are often stopped short at either Southsea or Fratton. On the 17th Sept, there were 168 cancelled services at Harbour, of which 149 instead started or terminated at either of the other stations. This suggests quite the capacity bottleneck on the pair of lines running into the station.

Interestingly, this data includes a mix of less-used rural lines alongside high-traffic commuter routes, although the former certainly accounts for more of the stations in the top 50 list.

## Inconsistent performers

This category looks at stations with a high standard deviation - i.e. variability in the cancellations they receive. In this data we can see a few familiar routes, the Heart of Wales line still holds firm at the top (it seems to be either totally cancelled or not at all - that'll be the engineering works!). There are a few newcomers to the top 25 though, with the TfW line to Caerphilly having several of its stations appearing in the top 20.

Other notable routes include:

- **The Northern City Line** (branch of Great Northern) to Moorgate: all stations on the branch from Finsbury Park appear in the top 25. Moorgate itself managed a 75% cancellation rate on 12th September!
- **Many other Great Northern stations** also feature, on both the Welwyn and Hertford sections of the line into London
- **The TfW branch line** from Llandudno to Blaenau Ffestiniog

## What does this data tell us? 

Despite what it may seem when you're travelling, none of the largest stations in the country feature on any of these lists. Cancellations seem to disproportionately affect a single route, rather than a single station.

Perhaps most encouragingly, nearly 20% of the network operates with perfect reliability, suggesting that consistent performance is achievable even at high-volume stations like Huddersfield. Perhaps the challenge isn't whether reliable railways are possible - it's extending that reliability across problematic corridors.

It'll be interesting to re-run this analysis in the future and see if we can weed out inconsistencies caused by engineering works and other freak incidents.