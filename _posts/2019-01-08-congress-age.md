---
layout: post
title: "How old is congress?"
author: "Brenton McMenamin"
tags: [demography]
---

As a companion to my [previous post]({{ site.baseurl }}{% post_url 2017-11-19-census %}) that combined several census datasets together to show the US age distribution over time, I thought I'd look at how the age of congressional representatives has changed over time as well.

GovTrack has provided a [data.world dataset](https://data.world/govtrack/us-congress-legislators) that contains the term dates for of each senator and house member in US history and birthdays for the vast majority of them.
With some simple [pandas-munging](https://github.com/bmcmenamin/sundries/blob/master/congress_age_distro/Congress%20Ages.ipynb), I was able to plot the distribution of ages for each chamber of congress.


The panels shows the age for each member of the senate (upper panel) or house (lower panel) every year. The solid black line is the median age.


<br>
<div align="center">
    <img alt="The average age of congressional members over time" src="/figs/cong_age_distro/congressional_ages_over_time.png" width="1200px">
</div>
<br>


The senate started getting older in the later half of the 19th century, dropped a little in the sixties, and then has been getting continually older from 1980 on as baby-boomers have started getting elected. The pattern looks similar in the house, but it's easier to isolate the period of age-change given the more frequent elections. In the house we see that age climbs from 1860-1876 and 1918-1932, a steep drop-off at the onset of the great depression, and then a steady climb from 1980 until the present.

