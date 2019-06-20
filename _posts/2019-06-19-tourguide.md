---
layout: post
title: "Combining geo-search and graph traversals on Wikipedia to find interesting stuff nearby"
author: "Brenton McMenamin"
tags: [releases, math&data]
---

Here's a new project, call [TourGuide](https://github.com/bmcmenamin/tourguide), that was inspired by my love of reading various [historical markers](https://www.hmdb.org/).


Here's what it does. You give it two inputs -- your latitude/longitude and a list of wikipedia pages for things that you're interested in. Then it does a geo-search of wikipedia to find things near your location, and starts a graph traversal to find pages that connect your current location to the things you're interested in. Looking over the results is like having custom historical markers that will teach you how the things around you are connected specifically to the things that you find interesting.

Give it a try [here](https://coastal-epigram-162302.appspot.com/).

Setting this up took a lot longer than expected due to several dead ends brought on by my unwillingness to pay real money to host a non-monetized side project.

The first attempt used [graphipedia](https://github.com/mirkonasato/graphipedia) store a wiki-dump in a Neo4J database for the graph traversals. But hosting a Neo4J wiki database was too expensive to a side project that I'm never going to monetize. Then, inspired by [Six degrees of Wikipedia](https://github.com/jwngr/sdow), I set up a SQL database with the wikipedia dump but it wasn't any cheaper to maintain. Finally, I set up the code to just use the existing [Wikipedia/Wikimedia APIs](https://www.mediawiki.org/wiki/API:Main_page). It's slower, but it's always up to date and I don't have to pay for any hosting\*! And running with Google App engine means that I'm only paying when instances get spun up rather than a 24/7 app.


\*I could speed up a lot of the searches if I cached API responses, but setting up memcache costs money. So, no.
