---
layout: post
title: "Mind the Data Gap: Interactive Delays"
categories: [Railways, Analysis]
tags: [railway, analysis, visualization, darwin]
---

# Part 4: Interactive Delays

Second only to cancellations, delays are a frustrating reality of the UK rail network. We've all got a story of a train delayed by an hour, or just enough to miss a connection. It's infuriating. But how common are delays in reality? We often hear statistics bandied about on the news, but how accurate are they? Let's dive into the world of analysing UK rail delays and see what we can uncover.

To obtain the data, we built a query that takes all active passenger services over the past 6 months and computes the delay between their latest registered schedule and the actual times they ran. This gives us delay data for each station on a service's route. We can then compute the average, min and max of the delays across the stations along the route.

## Delay Distribution Over Time

The first visualisation is a stacked area chart, displaying the percentage of services that fall into each delay bucket over time. Use the dropdown to filter by Train Operating Company (TOC) or view aggregated data across all operators.

<iframe src="/assets/visualizations/delayed-services-stacked-area.html" width="100%" height="750" frameborder="0" style="border: 1px solid #ddd; border-radius: 8px; margin: 20px 0;"></iframe>

## TOC Performance Comparison

The second visualisation uses a bubble chart to compare performance across different train operators. Explore the relationship between average delays, consistency, and the percentage of delayed services. Bubble size represents the proportion of services delayed beyond your selected threshold. You can filter by date range and delay threshold to dive deeper into the data.

<iframe src="/assets/visualizations/delayed-services-bubble.html" width="100%" height="1200" frameborder="0" style="border: 1px solid #ddd; border-radius: 8px; margin: 20px 0;"></iframe>

## Key Findings
From these visualisations, several patterns emerge:

**Temporal Patterns:**

- Rail performance is generally consistent day-to-day, though there are notable drops on Sundays — likely owing to the fact most rail staff are not contracted to work on Sundays.
- In most cases, the more consistent an operator is (lower standard deviation), the less delays they have. The only outlier to this is the Caledonian Sleeper — although this is likely due to the small amount of services they run.

**Operator Performance:**

- Certain train operators (Avanti, CrossCountry) have notably poorer performance than other operators, with higher average delays and greater inconsistency.
- The best performers (Merseyrail, Heathrow Express, Island Line) maintain both low average delays and high consistency.
- The Eurostar has terrible punctuality!

**Overall Picture:**

While this might not give you much comfort when standing on a cold train platform, it seems most trains run on time! The best performers see 95% or more of their services running within 5 minutes of schedule. I just hope you aren’t waiting for an Avanti or Eurostar service, as you might be waiting a while…