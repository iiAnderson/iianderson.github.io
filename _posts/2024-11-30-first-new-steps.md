---
layout: post
title:  "First new steps - Analysing Darwin data"
categories: [Railways, Analysis]
tags: [railway, analysis, sql, darwin]
---

As mentioned in my first article, over the last few months I've been collecting data from National Rail's Darwin push port - a dataset containing all train movements across the UK. In these articles I intend to explore this dataset further, hopefully pulling out some interesting insights for both the train enthusiast, commuter and government alike. 

This first article I want to start with a simple bit of analysis - but one that I was suprised by. But first, let's explain what the data we'll be looking at is.

In the National Rail systems, there is a considerable nuance to the definition of a rail service, but generally each service will have an origin and a destination, with some intermediate locations in between the two termini. These locations are identified by `tpl` codes in the datasets which encode TIPLOC values (lookup here). A service is identifed by an `rid`, which is a unique identifier for that service on that given day.

For our first bit of analysis, we will analyse the number of unique passenger services that pass through each station in the UK, ordering by largest to smallest. This will provide us a table of the the number of `rid`s passing through a given `tpl`.

And this is what we get - note this is data for the 22nd November.

tpl	   count
LNDNBDE	1302
LEEDS	1237
WATRLMN	1212
WATRLWC	1212
STFD	1160
LIVST	1018
BTHNLGR	1015
MNCRPIC	1006
ARDWCKJ	978
WDON	944
ECROYDN	943
RAYNSPK	943
WNDMLBJ	940
CLPHMJM	935

