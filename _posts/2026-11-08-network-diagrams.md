---
layout: post
title: "Generating network diagrams for UK stations"
description: "Building network diagrams for all UK stations"
categories: [Railways, Data]
tags: [railway, dataset, aws, open-data, darwin, visualisation, documentation]
---

I've always loved a rail network diagram. I remember spending hours as a child staring at the Network SouthEast map at my local station. It fascinated me to see which operators stopped where - and I was always proud that my local town, Worthing, had a bold circle meaning __interchange__.

What was missing from those maps was the routes the trains took - I could see Southern services stopped - but which ones? Where could I get to without changing? It was this mission that first set me on trying to generate these maps - and has led to this blog post.

The visualisation below shows all stations with 2 or more trains per hour, and what routes call at those stations. To do this, I utilised the [data collected from National Rail](/posts/a-new-dawn-in-uk-rail-analysis/) to understand which services called at a station, grouped these into rough routes, and displayed them. It certainly starts getting interesting when you look at some of the larger stations! The numbers shown by stations are the trains per hour from the selected station.

<iframe src="https://rail.robbiea.co.uk/routes" width="100%" height="1000" frameborder="0" style="border: 1px solid #ddd; border-radius: 8px; margin: 20px 0;"></iframe>

## How I did it - the detail

If you're interested to learn more about the journey for generating these - read on!

We start by collecting the data. To do this, I queried my [normalised dataset](/posts/the-darwin-normalised-dataset/), fetching all the services calling at a given station. This elicited a lot of routes, as the same services in different hours often have slightly different calling patterns. This required some grouping - routes where all services almost matched were grouped together. 

From here, I had a list of grouped services for each station. These now needed to be graphed.

### Graphing - tree

Boiling a network diagram to its core: the edges are the route a service takes to get between two consecutive stations. As we are displaying routes, a line doesn't correlate to physical track - several lines can share one track. 

This sounds like a tree! If we take every service through the hub, join consecutive stops into a graph, and keep a maximum-weight spanning tree of that graph, weighted by trains per hour, we get a tree for that given station.

To ensure the tree doesn't criss-cross, branches leave each station at 45° angles, as close to their real compass bearing as possible. Sibling branches keep this angular order, so they never swap sides and cross. Within a shared corridor, each route keeps its lane, and lanes are ordered by where each route goes next, so lines only change side where they split.

This algorithm was repeated several times per station, measuring route crossings, label collisions and turn-backs. The attempt with the fewest failures won.

There were a few issues with this approach though. A tree can't contain loops, so some links between stations had to be dropped to build one. This meant the diagram couldn't account for loops, or cases where there are multiple paths to a station. The tree then showed a station in multiple places or created fake termini where it thought a route ended.

![Tree layout of the Horsham network diagram](https://blog.robbiea.co.uk/assets/img/tree-failure-horsham.png "The tree layout at Horsham")

![Tree layout of the Sheffield network diagram](https://blog.robbiea.co.uk/assets/img/tree-failure-sheffield.png "The tree layout at Sheffield")

### Graphing 2 - blocks

Instead of approaching the graph as a tree, I focused on the loops first. I split the graph into the parts that contain loops, where you can lose any one link and still get between stations another way, and the parts that don't: branches and dead ends. The loop-free parts are drawn as spokes, exactly as in the tree approach.

Inside a loop section, any run of stations without a junction collapses into a single corridor - that's easy. That leaves a much smaller problem: a handful of junctions and the corridors between them. The corridors leave each junction in their real compass order, so the optimiser only decides shapes, never which line goes where. It then places the junctions and sizes the corridors together, so every loop closes and every corridor has room for its stations. Among the layouts that work, it picks the one with the fewest crossings, then the fewest bends, then the smallest deviation from real bearings.

#### Real data: most failures were in the graph

When I first ran the blocks approach on real stations, a lot of the maps looked wrong. 

But most of the bad drawings didn't come from the layout. They came from the graph. A fast train only tells you where it stops, so I filled in the stations it passes using other services' stopping patterns. When that fill-in got it wrong, the map showed a link that doesn't exist, and that fake link often created a loop that doesn't exist either.

![Balham before and after fixing the graph](https://blog.robbiea.co.uk/assets/img/blocks-balham-before-after.png "Balham, before and after fixing the graph")

- **Detours.** The fill-in borrowed the longest stopping pattern it could find, so Leeds to York went round the Harrogate loop. 
- **Gaps.** The Gatwick Express borrowed a semi-fast's stops, but the semi-fast itself skips a stretch. The result was a fake link that turned Clapham Junction into a junction and drew it south of Balham. 

#### Real data: rendering

With the graph fixed, the remaining problems were in how lines were drawn.

- **Lanes jumping at junctions.** When a route turned off, the remaining lanes re-centred themselves and crossed each other. At Clapham Junction, the whole 10-lane main line shifted sideways where the Windsor lines left, and at Wivelsfield (below) a branch's lanes crossed the routes going straight on.
- **Stop dots off their route.** Interchange dots were spaced evenly across a station without checking where the lines actually ran. At Clapham Junction, 64 of 388 dots sat off their route - which isn't great!
- **Sprawl.** The search kept picking the widest spacing for tiny score gains. Now it takes the tightest spacing within a few points of the best. This shrank Clapham Junction from about 9,300 × 11,900 to 3,900 × 5,300!

![Wivelsfield before and after fixing lanes at junctions](https://blog.robbiea.co.uk/assets/img/block-wivesfield-before-after.png "Wivelsfield, before and after fixing lanes at junctions")

Now these diagrams aren't perfect - especially on the complex stations - but they're a start. I've loved looking through these diagrams and seeing where services run. It really reminds me of the complexity and sheer scale of our railway. I hope you get the same enjoyment.
