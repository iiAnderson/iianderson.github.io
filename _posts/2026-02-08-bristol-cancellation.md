---
layout: post
title: "Half of Bristol train cancellations are due to one reason"
categories: [Railways, Analysis]
tags: [railway, analysis, visualization, darwin, bristol]
---

# Half of Bristol train cancellations are due to one reason

We've all experienced train cancellations - leaves on the line, signalling faults, the list is seemingly endless. Turns out it's not though, National Rail actually has 506 different cancellation codes which cover every eventuality. But when I analyzed December's data from Bristol Temple Meads, one pattern dominated: nearly half of all cancellations came down to staffing shortages.

Analysing the raw National Rail data, we can see the top 10 delay categories were:

| Count | Cancellation Reason |
|-------|---------------------|
| 145 | This train has been cancelled because of a shortage of train crew |
| 72 | This train has been cancelled because of more trains than usual needing repairs at the same time |
| 67 | This train has been cancelled because of a shortage of train drivers |
| 21 | This train has been cancelled because of a fault on this train |
| 20 | This train has been cancelled because of severe weather |
| 16 | This train has been cancelled because of a fault with the signalling system |
| 13 | This train has been cancelled because of a broken down train |
| 12 | This train has been cancelled because of a points failure |
| 9 | This train has been cancelled because of the emergency services dealing with an incident |
| 7 | This train has been cancelled because of train crew being delayed by service disruption |

Some of these categories are quite similar, if we group similar reasons together, we get:

- Related to a shortage of crew: 212
- Related to the trains (faults/repairs): 106
- Related to infrastructure (signalling, points): 28
- Related to weather events (flooding): 20
- Related to previous delays: 7
- Related to emergency services: 9

This is a striking finding: 46% of cancellations were due to staffing decisions by train operating companies, not infrastructure failures or weather events. Imagine what an effect this could have on our railways - and this doesn't even include delays caused by shortage of crew!

Through this analysis, we've also captured when these services were cancelled, as shown below.

| Cancellation time | Count |
|-------------------|-------|
| Cancelled: <1 hour before departure | 123 |
| Cancelled: 4+ hours before departure | 120 |
| Partially cancelled (service ran partway) | 113 |
| Cancelled: 2-4 hours before departure | 51 |
| Cancelled: 1-2 hours before departure | 49 |

We have a broadly even split between < 1 hour before departure and 4+ hours. This suggests in most cases, trains are cancelled last-minute or far ahead of time. We were also able to compute services which partially ran - i.e. they stopped at Temple Meads but started their journey later or terminated earlier than scheduled.

Comparing these cancellation times to the cancel reasons shows largely what we'd expect (see below). 40% of crew shortage cancellations happen 4+ hours from departure, suggesting it was known at the start of the day of the lack of staff. Train faults take up nearly 50% of the hour before departure cancellations, where trains are failing during service and causing disruption.

<iframe src="/assets/visualizations/cancellations-bristol-december.html" width="100%" height="950" frameborder="0" style="border: 1px solid #ddd; border-radius: 8px; margin: 20px 0;"></iframe>


In the future, it would be interesting to see how the train fault stats compare against other stations, as Bristol is served by Diesel traction only - which is generally less reliable than electric traction.

So, the next time your train is cancelled, remember: it's not always leaves on the line. The data suggests that better crew planning could prevent nearly half of UK rail cancellations.