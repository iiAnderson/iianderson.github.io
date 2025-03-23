# Analysing select services out of Bristol Temple Meads

Over the past few months, I have been designing a system for ingesting data from National Rail’s “Darwin Push Port Feed”. This feed contains live information about train movements, scheduling, and service alterations across the UK rail network. In this article, I will share some analysis of the service delays for three most popular (by passenger numbers) routes out of Bristol Temple Meads. These are the route to London Paddington, Cardiff Central and Clifton Down.

## The Query

To obtain this data, I used the query found in the appendix, it does a few things of note:
- It fetches the last SCHED and ACT (scheduled and actual) messages for each tpl. In this case, it's assumed the last schedule message is the most accurate.
- It attempts to correct for day rollover (where a service runs into the next day) when computing the delay for a service. This is done by comparing the schedule and actual arrival times, and if the difference is off by more than half a day (I assume no service is this late!) then it adjusts the calculation to ensure the correct delay is calculated.
- It assumees that the last and first locations updates are the destination and origin repsectively. This could be incorrect if there is data missing (ie I fail to capture the start of a service).

This query was then run for each three routes, pulling back the raw data into Google Sheets for analysis. 

## The Analysis

To analyse the data, I needed to ensure only the correct data was being pulled in, as my query cannot currently understand directionality of a route. For example, a service can be arriving in Bristol Temple Meads from either Weston (en route to Paddington) or from Paddington (either terminating or continuing to Weston). This was done by filtering the data for only services that started in the correct direction - in this example, if I want to analyse services going to London Paddington via Bristol, they would need to terminate at either Taunton, Weston or the South West of Wales.

Once that's done, we can see some lovely heat maps!

### Services to London Paddington

![Alt text](https://blog.robbiea.co.uk/assets/img/BRI-PAD-March25.png "Bristol to Paddington delays by hour across march")

### Services to Cardiff Central

![Alt text](https://blog.robbiea.co.uk/assets/img/BRI-CDF-March25.png "Bristol to Cardiff delays by hour across march")

### Services to Clifton Down

![Alt text](https://blog.robbiea.co.uk/assets/img/BRI-CFN-March25.png "Bristol to Clifton Down delays by hour across march")