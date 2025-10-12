---
layout: post
title: "Mind the Data Gap: Rush Hour Reality"
categories: [Railways, Analysis]
tags: [railway, analysis, visualization, darwin]
---

# Part 4: Interactive Delays

# Part 5: Rush Hour Reality

Over the last year, I have been designing a system for ingesting data from **National Rail’s “Darwin Push Port Feed”**.  
This feed contains live information about train movements, scheduling, and service alterations across the UK rail network.  

In previous posts, we have discussed how services can be grouped into routes, analysed stations for the most and least delayed, and developed interactive visualisations to show delays across the network.  
Today we’re going to focus on **peak trains**, their **capacity**, and their **average loading** — a metric to quantify how full a given service is.  

Loading is a relatively new metric to the Darwin system, being first introduced in 2017 ([1] yes, the railways move slowly...) and has been slowly adopted by operators.  
Over the course of the last six months, we’ve only been able to capture reliable data for both **Chiltern** and **Southeastern**.

---

## Loading Distribution

To conduct the analysis, we need to pull all services over a given period and compute the average loading for each of them.  
This gives us the following loading distribution (0 is empty, 100 is full including standing):

| Loading Range | Records | Percentage |
|----------------|----------|-------------|
| 0–20% | 106,285 | 56% |
| 20–40% | 51,212 | 27% |
| 40–60% | 17,588 | 9% |
| 60–80% | 8,141 | 4% |
| 80–100% | 6,485 | 3.4% ← **Severe overcrowding** |

Perhaps unsurprisingly, **trains are often very empty!**  
Just 7% of services are running at full seating capacity.

---

## Average Loading by Time of Day

When we group these by time (across a sample at the start of October), the loading factor increases drastically during the early morning and evening peaks:

| Time Period | Average Loading | Notes |
|--------------|-----------------|--------|
| 4–5am | 10% | |
| 5–6am | 18% | |
| 6–7am | 24% | Early commuters |
| 7–8am | 35–37% | Morning peak starts |
| 8–9am | 34–36% | Morning peak |
| 9–10am | 21% | |
| 10am–2pm | 18–20% | |
| 2–3pm | 21% | |
| 3–4pm | 31–32% | School run |
| 4–5pm | 33–34% | Evening peak starts |
| 5–6pm | 29% | Evening peak |
| 7–8pm | 20% | |
| 8pm–midnight | 13–17% | |

---

This pattern doesn’t continue at the weekends though, where instead we see:

- **No morning peak:** 7–8am only 14% (vs 36% weekday)  
- **Midday peak instead:** 10am–12pm reaches 23–25%  
- **Flattened evening:** 5–7pm only 24% (vs 33% weekday)  
- **Later activity:** 8pm–midnight stays higher (16–19% vs 13–17%)

---

## Fleet Composition

There are a few other factors outside of time of day that affect service loading — these being **frequency** and **length** of the service.  
We won’t be touching on frequency today, but let’s explore service length and its effect on loading.

Let’s start with a simple analysis on the length of services across the two operators:

### Southeastern Fleet

```
2–4 cars:  ██████████ 14.1%
5–7 cars:  ████████  9.1%
8 cars:    ████████████████████████████████ 48.9%  ← BACKBONE
10 cars:   █████████████ 21.0%
12 cars:   ████ 6.6%
```

### Chiltern Fleet

```
2 cars:    ████████████████ 24.9%
3 cars:    ███████████████ 23.9%
4 cars:    █████████████████████████ 39.1%  ← BACKBONE
5–8 cars:  ██████ 11.2%
```

Despite both operators running commuter services into London, the above shows the disparity in rolling stock they have available,  
with **Southeastern’s trains being far more fit-for-purpose** on the busy commuter routes.

But do the shorter services of the Chiltern lines affect its overcrowding at peak times?  
If we group our services by length, then compute the average loading factor, we can get a feel for whether length is affecting overcrowding of peak services.

---

## Peak-Time Overcrowding by Train Length

During peak times, we get the following table:

| TOC | Operator | Train Length | # Services | Avg Loading | Min–Max | Severity |
|-----|-----------|---------------|-------------|--------------|----------|-----------|
| SE | Southeastern | 5 cars | 14 | 80.9% | 65–98% | 🔥🔥🔥 **CRITICAL** |
| SE | Southeastern | 10 cars | 38 | 73.7% | 61–93% | 🔥🔥 **SEVERE** |
| SE | Southeastern | 12 cars | 1 | 71.5% | 71–71% | 🔥🔥 **SEVERE** |
| SE | Southeastern | 8 cars | 8 | 69.7% | 60–81% | 🔥🔥 **SEVERE** |
| SE | Southeastern | 3 cars | 3 | 70.1% | 60–78% | 🔥🔥 **SEVERE** |
| SE | Southeastern | 6 cars | 5 | 65.0% | 62–68% | 🔥 **HIGH** |
| CH | Chiltern | 4 cars | 10 | 67.2% | 60–88% | 🔥 **HIGH** |
| CH | Chiltern | 3 cars | 6 | 68.8% | 62–76% | 🔥 **HIGH** |
| CH | Chiltern | 5 cars | 2 | 65.6% | 62–69% | 🔥 **HIGH** |
| CH | Chiltern | 2 cars | 1 | 61.3% | 61–61% | 🔥 **HIGH** |

---

14 of the services running during peak times for **Southeastern** are **5-car trains** — with an average loading between **65% and 98%**,  
accounting for nearly **20% of the overcrowding at peak**.  
A rebalancing of trains during peak times could drastically cut this congestion.  

The same could also be said for the **10-car services**, which are equally stretched during peak times.  

Looking closer at the data for **Chiltern**, we can see a different picture, with the **longest trains having the largest loading factor**.  
These services are only **4 cars long**, something which if increased could have a drastic improvement to passenger comfort at peak.

You can play with this data further in the visualisations below

---

## Interactive Visualizations

### Loading Distribution

Explore the distribution of train loading across all services, showing how often trains fall into different capacity ranges:

<iframe src="/assets/visualizations/rush-hour-loading-distribution.html" width="100%" height="750" frameborder="0" style="border: 1px solid #ddd; border-radius: 8px; margin: 20px 0;"></iframe>

### Hourly Loading Pattern

See how train loading varies throughout the day, with clear peaks during morning and evening rush hours:

<iframe src="/assets/visualizations/rush-hour-loading-hourly.html" width="100%" height="750" frameborder="0" style="border: 1px solid #ddd; border-radius: 8px; margin: 20px 0;"></iframe>

---

### Reference
[1] [Celebrating 20 years of Darwin – The railway's single source of truth](https://railuk.com/travel/celebrating-20-years-of-darwin-the-railways-single-source-of-truth/)
