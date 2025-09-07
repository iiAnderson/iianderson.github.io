---
layout: post
title:  "Mind the Data Gap: Part 2. Grouping Routes"
categories: [Railways, Analysis]
tags: [railway, analysis, sql, darwin]
---

## Part 2: Grouping Routes

Over the last year, I have been designing a system for ingesting data from National Rail’s “Darwin Push Port Feed”. This feed contains live information about train movements, scheduling and service alterations across the UK rail network. Today we’re going to delve into my first visualisation attempt for this data, a page which displays all routes from a select number of ‘Hub’ stations across the country, and strategies for grouping these together.

The Darwin dataset is complex: it holds information on every service across the entire country for any given day. So it’s no surprise it took me a while to decide where to start visualising it \- should I focus on delays? Or service patterns? It was CrossCountry (the franchise) that nudged me toward *routes*, thanks to their cancellation of the Penzance-to-Aberdeen service. It made me wonder \- how many other wacky long-haul services are hidden among our daily routes?

The visualisation began, as all good things do, with data collection. I needed details for every service across the country: origin, destination and calling points. This quickly became impractical. Darwin includes heaps of non-passenger services, often tricky to identify, which meant lots of manual filtering before I conceded defeat. So I narrowed the scope and focused on collecting service information from a single hub station, cutting down the data dramatically.

Turns out there are *quite a lot* of trains on the British rail network, and visualising them all from a hub like Bristol Temple Meads wasn’t exactly user-friendly. Unless you’re the Terminator (and if you are, why are you reading this blog?), the result was just a mess of squiggles on a map. I needed a better way to show routes rather than individual services. To do that, I first needed a solid definition of a “route.”

My initial definition of a route was “a collection of services which start at/terminate at similar locations, and follow a similar calling point pattern”. Now that’s pretty vague, and any human or (especially) AI is going to read that and do a terrible job of implementing it in an algorithm, so I needed a more refined definition.

I began in what felt like the most sensible place: grouping by origin and destination. This worked fairly well for smaller stations like Worthing, where services follow a neat south-coast pattern. But it fell apart at larger hubs serving many long-distance routes. Worthing ended up with 11 routes; Bristol Temple Meads ballooned to 43\.

Here’s why Bristol exploded in route count. Trains to London Paddington often start at Temple Meads, once an hour in fact. But the *other* hourly service begins at Weston-super-Mare. Under the origin/destination rule, these weren’t grouped together. Add the four daily services starting at Taunton instead of Weston, and then toss in the Exeter/Penzance starters, and suddenly we’ve got four “routes” where we really wanted just one.

To combat this, I attempted to use a new strategy, which I christened *Corridor Similarity*. This attempted to group service patterns together if the origin/destinations were within a few stops of each other (practically this included PASSing locations as well as stopping points). This then meant there was a tolerance allowed to group services which started/finished in similar locations. This algorithm also then went on to assess how much of the route was shared between the services. If this was above a certain percentage, the services were grouped together. This actually worked pretty well, grouping the aforementioned London services together, still didn’t group any of the long-distance cross country services running from Exeter through Bristol to Birmingham.

Another strategy again was needed. A train’s headcode is an identifier for a given service. It’s structured as the following 4 characters:

- A single digit indicating the class of train, such as express passenger, stopping or freight  
- A character indicating the destination area (although this varies, as we’ll explore below)  
- Two digits representing the individual service, usually incrementing by route across the day

*See: [https://live.rail-record.co.uk/headcode/](https://live.rail-record.co.uk/headcode/) for more information.*

This theoretically gives us information on the type of service, as well as the region/route the service is running—perfect for grouping. In practice, of course, it’s not that simple. The second character, meant to identify the region/route, isn’t well standardised. If a service leaves its region, it takes on the code of its destination region. If it stays local, almost any code can be assigned. For instance, Brighton–Seaford trains use a “C,” but a completely different service elsewhere could use that same character.

While a pain, this doesn’t affect our grouping logic. We can get the headcode for each service and group by the first two characters (or just the second character if we want to), giving us the routes from a given hub station. Through my testing, this has been the most reliable grouping mechanism, consistently grouping similar services whilst minimising what I would deem as “incorrect” groups.

You can play with the visualisation and grouping mechanisms on my site here.

