---
layout: post
title: "Service Delay Distribution: An Interactive Analysis"
categories: [Railways, Analysis]
tags: [railway, analysis, visualization, darwin]
---

# Service Delay Distribution: An Interactive Analysis

Over the period from September 11-25, 2025, we captured detailed delay data across the UK rail network. This visualization shows how services are distributed across four delay buckets: 0-5 minutes, 5-15 minutes, 15-30 minutes, and 30+ minutes delayed.

## Interactive Visualization

Use the dropdown below to filter by Train Operating Company (TOC) or view aggregated data across all operators.

<iframe src="/assets/visualizations/delayed-services-stacked-area.html" width="100%" height="650" frameborder="0" style="border: 1px solid #ddd; border-radius: 8px; margin: 20px 0;"></iframe>

## Key Findings

The stacked area chart reveals several interesting patterns:

- **Overall Performance**: The majority of services across the network fall within the 0-5 minute delay bucket, suggesting generally good performance
- **TOC Variations**: Different operators show distinct delay patterns when filtered individually
- **Temporal Patterns**: Certain dates show spikes in longer delays, possibly corresponding to specific incidents or weather events

## About the Data

This analysis is based on data from National Rail's Darwin Push Port Feed, covering:
- **Time Period**: September 11-25, 2025
- **Services Analyzed**: Thousands of individual train services
- **TOCs Covered**: All major UK train operating companies

The percentages shown are weighted by the number of services, ensuring that high-volume operators are accurately represented in the "All TOCs" view.
