---
layout: post
title: "'Dear Abby' for data science"
author: "Brenton McMenamin"
---

I've always liked advice columns because I'm jealous that there's some individual whose entire job description is "be opinionated about other people's problems".

Recently, I've also realized that advice columns are also a potentially great dataset for tinkering with text analysis algorithms. The main draw is that each article is structured in a Question-Answer format that would allow for testing of sequence-to-sequence learning or clustering of questions and answers into topics.

So, I scraped together the daily advice columns written by [Dear Abby](https://en.wikipedia.org/wiki/Dear_Abby) from 1991 to 2017. The dataset (and the scripts used to make it) live [here](https://github.com/bmcmenamin/word2vec_advice/tree/master/scrape_scripts).

Some quick background: _Dear Abby_ is an English-language advice column that was first published is newpapers starting in the 1950's. People write in to get help from Abby, and she publishes your problem along with her advice on what to do. I do not own the content of Dear Abby. All rights reserved to Andrews McMeel Universal.