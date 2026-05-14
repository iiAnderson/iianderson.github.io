---
layout: post
title: "Londoners wasted 8 lifetimes waiting for cancelled trains in 2025"
categories: [Railways, Analysis, London]
tags: [railway, analysis, visualization, london, glasgow, cancellations, uk]
---

## Glasgow tops the cities. A small town in Sussex tops everywhere.

I have been collecting train service data from National Rail — storing each train's movements, delays and cancellations. I am now working on developing new analysis and visualisations to show this data, along with holding the bodies that run our railways accountable. In the last article, we explored the number of cancellations each station has seen over the last year. Today we're going to be diving into cancellation impact, and finding that while Londoners waited nearly 8 lifetimes for cancelled trains, there are towns and cities that feel the impact more acutely.

## The initial analysis

Let's start by computing the stations with the most cancellations in 2025. Using our normalised table (see previous blog) we can quickly calculate this, aggregating across each station and grouping by CRS code. This gave us our top ten stations by percentage of services cancelled, which contains some stations from rural branch lines (the Conwy Valley line in Wales to Blaenau Ffestiniog), but also contains the well-used Sutton Loop run by Thameslink — certainly not a rural backwater!

## The quest

While percent of services cancelled shows part of the story, it doesn't paint the entire picture. A few cancellations on rural lines will have an outsized effect on the percentage of services cancelled — as some of these lines run as few as five trains a day! Ideally, we would also include the number of passengers — as this would highlight where a cancellation has more of an impact.

Getting this data would be tricky, though. Only some train companies publish the loading of their services (how full a train is, as a percentage) and even then they don't publish numbers for people standing at the station waiting for a cancelled train. So how can we work this out? By using our good friend approximation!

## The new dataset

Each year, Network Rail publishes an Origin-Destination Matrix, containing data on how many people bought tickets between every pair of stations in the country. Want to know how many people bought tickets from Penzance to Thurso? Only two people were mad enough to do this!

From this dataset, we can then approximate how many people travelled between two stations on a given day by dividing the annual figure by 365. This is a very rough figure — it doesn't take into account the stations the services visit, peak times or holidays — but it will work for now!

## The quest — intermediate stops

The ODM dataset is not a silver bullet, though. We're still going to have to do some work to make it useful. Unless you're travelling on the future HS2, trains tend to stop in more than one location. This makes our computation a little more complex. Let's say we had a train running from Bristol to Bath, calling at one intermediate stop, Keynsham. If the service was cancelled outright (i.e. never ran), we would need to compute the passengers affected from Bristol → Keynsham, Bristol → Bath and Keynsham → Bath. We need to ensure we compute affected passengers for all permutations (moving forward from the point of origin) of the service stops — not just the origin itself.

Let's work through it! Let's say:

- 100 people travel between Bristol → Keynsham daily
- 500 people travel between Bristol → Bath daily
- 200 people travel between Keynsham → Bath daily
- There are 10 services which run this route daily (stopping at all three stations)

If one of our services is cancelled outright, then we would need to compute the number of people we expect to be on the service. This would be:

- 100 / 10 = 10 people travelling from Bristol → Keynsham
- 500 / 10 = 50 people travelling from Bristol → Bath
- 200 / 10 = 20 people travelling from Keynsham → Bath

Which gives us *80* affected passengers!

If a service was only partially cancelled (e.g. the train ran part of its route but was cancelled for some stops), only the stops that were actually cancelled are counted. Stops where the train did run are ignored.

## The results — initial

We have our algorithm, we have our data — let's do some computation! And our winner is... Glasgow Central, closely followed by Glasgow Queen Street.

| Station Name | Affected Passengers |
| --- | --- |
| Glasgow Central Rail Station | 1,883,590 |
| Glasgow Queen Street Rail Station | 1,805,403 |
| London Bridge Rail Station | 1,561,119 |
| London Waterloo Rail Station | 1,463,546 |
| London Victoria Rail Station | 1,288,871 |
| London Paddington Rail Station | 1,144,402 |
| Stratford (London) Rail Station | 1,053,249 |
| Farringdon (London) Rail Station | 1,044,242 |
| London Euston Rail Station | 1,036,101 |
| London Liverpool Street Rail Station | 1,029,034 |

We have a lot of the culprits we'd expect in the top 10 — busy central London stations that carry a huge passenger volume. Interestingly, there's only one station here on the Thameslink core, despite its reputation as the most unreliable line in the country.

It's also worth noting that the three top London stations are the largest termini serving the commuter belts south of London, where congested lines often lead to delays and cancellations, although London Bridge's high passenger impact is partly self-inflicted — staff shortages account for about 7 cancellations a day.

We also see both central Glasgow stations. It has a wide suburban network that has a lower frequency than other major cities, so a single cancellation has a higher number of passengers affected (~460 per service from Glasgow Queen Street). This lends weight to the argument that these services are underfunded, and could do with investment to reduce delays and increase frequency — taking them closer to their London counterparts.

## The results — grouping by cities

If we aggregate the top 30 stations by city, we can measure how affected each city's residents are by cancellations. Doing so gives a top 10 of:

| City | Number of Stations | Affected Passengers | Metro Pop (approx) | Cancellations Per Person |
|------|-------------------|---------------|-------------------|------------------------|
| Glasgow | 2 | 3,688,994 | 1,200,000 | 3.07 |
| London | 19 | 16,076,838 | 8,800,000 | 1.83 |
| Brighton | 1 | 633,256 | 480,000 | 1.32 |
| Reading | 1 | 362,645 | 320,000 | 1.13 |
| Cardiff | 1 | 433,403 | 500,000 | 0.87 |
| Leeds | 1 | 581,535 | 790,000 | 0.74 |
| Edinburgh | 1 | 435,459 | 900,000 | 0.48 |
| Liverpool | 1 | 732,708 | 1,550,000 | 0.47 |
| Birmingham | 1 | 914,517 | 2,900,000 | 0.32 |
| Manchester | 1 | 755,213 | 2,870,000 | 0.26 |

Glasgow still comes out on top with nearly *3 cancellations per resident per year*, but we do have a few new candidates, namely Brighton and Reading. These are major commuter towns and have a frequent service into London, so it's no surprise they feature heavily here. Of note is how low the large cities of Manchester and Birmingham are — both of which have well under 1 cancellation per person per year. This is very much at odds with Northern's reputation as one of the least reliable train operating companies. If Northern is really that unreliable, why isn't it showing up here? It's a thread worth pulling on in a future post.

What does this look like if we scale up to 100 stations? Below is a visualisation, showing Glasgow has been pipped at the post by *Haywards Heath*, a small town south of London halfway along the Brighton Main Line. Due to its proximity to the capital, it's a major commuter origin — with hundreds of services per day. It's also fairly removed from neighbouring towns, meaning driving and the train are the only realistic options for commuting. This is in stark contrast to cities such as Manchester, which have a well-run tram and bus network, easing the congestion on rail lines.

As Haywards Heath sits on the Brighton Main Line, which feeds into the Thameslink core, delays from Bedford, Cambridge, and Kent can ripple south, and the town sits squarely in the cascade zone.

<iframe src="/assets/visualizations/20260414-city-impact.html" width="100%" height="1000" frameborder="0" style="border: 1px solid #ddd; border-radius: 8px; margin: 20px 0;"></iframe>

So what does this tell us? It suggests some systemic issues with the services in and around Glasgow, with the city's stats showing cancellations are far worse than its counterparts — with only London coming close in terms of passengers affected per resident.

It's also interesting to see the effects on small commuter towns too — where cancellations have an outsized impact. If we perhaps assumed half of the residents of Haywards Heath get the train, that's roughly *one cancelled train every three weeks of working days* — your morning gets blown up almost monthly, and that's before you count the delays that didn't quite tip into cancellation. We can see this pattern repeated for other smaller towns too, such as St Albans with 2.4 cancellations per resident per year and Redhill with 3.

Finally, it shows the sheer amount of commuting happening across the capital every day — and the impact cancellations have on people's lives. If each of the 22 million affected passengers had to wait an extra 15 minutes for their train, that equates to *nearly 628 years lost to cancelled trains* — that's 8 lifetimes — just in London. A staggering amount.

In future posts, we will look to refine this algorithm and explore other avenues of analysis — such as investigating regional cancellation differences and how long-distance services affect cancellations.