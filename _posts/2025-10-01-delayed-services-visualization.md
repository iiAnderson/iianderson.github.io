---
layout: post
title: "Mind the Data Gap: Interactive Delays"
categories: [Railways, Analysis]
tags: [railway, analysis, visualization, darwin]
---

# Part 4: Interactive Delays

Second only to cancellations, delays are frustrating reality of the UK rail network. We've all got a story of a train delayed by an hour, or just enough to miss a connection. It's infuriating. But how common are delays in reality? We often hear statistics bounded around on the news, but how true are those statistics? Let's dive into the world of analysing UK rail delays and see what we can uncover.

To obtain the data, we need to build a query

## Interactive Visualization

Use the dropdown below to filter by Train Operating Company (TOC) or view aggregated data across all operators.

<iframe src="/assets/visualizations/delayed-services-stacked-area.html" width="100%" height="750" frameborder="0" style="border: 1px solid #ddd; border-radius: 8px; margin: 20px 0;"></iframe>

## TOC Performance Analysis

Explore the relationship between average delays, consistency, and the percentage of delayed services across all operators. Bubble size represents the proportion of services delayed beyond your selected threshold.

<iframe src="/assets/visualizations/delayed-services-bubble.html" width="100%" height="1200" frameborder="0" style="border: 1px solid #ddd; border-radius: 8px; margin: 20px 0;"></iframe>

## Key Findings

The stacked chart suggests:
- Rail performance is generally consistent, although there are notable drops on Sunday's - likely owing to the fact most rail staff are not contracted to work on Sundays.
- Certain train operators (Avanti, Cross Country) have notably poorer performance than other operators.
- Most trains run on time!
