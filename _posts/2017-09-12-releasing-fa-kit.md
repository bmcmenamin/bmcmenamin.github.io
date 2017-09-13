---
layout: post
title: "Launching the Factor Analysis Kit (fa-kit)"
author: "Brenton McMenamin"
---

Back in the day when I first used [Factor analysis](https://en.wikipedia.org/wiki/Factor_analysis) to explore multivariate datasets, it was already considered ancient (and that was nearly fifteen years ago...). Regardless, it's been an indispensable part of my workflow for a 
[variety](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC2871966/) [of](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC4138337/) [datasets](https://theblog.okcupid.com/the-8-personalities-youll-meet-when-dating-in-the-u-s-9d87a5a40274).

I've had to frequently re-write the same set of tools because factor analysis isn't sexxxy enough to get included in major python packages. Scikit-learn has a bunch of good tools for matrix decomposition -- and there is one named (FactorAnalysis)[http://scikit-learn.org/stable/modules/generated/sklearn.decomposition.FactorAnalysis.html#sklearn.decomposition.FactorAnalysis]! Unfortunately, it doesn't perform any matrix rotations so I still have to google stackoverflow to figure out that part.

So I sat down and organized all my scattered pieces of factor analysis code into the (Factor Analysis Kit (FA-Kit))[https://github.com/bmcmenamin/fa_kit] and decided to share it. It works with python 2 and 3 and is installable as `pip install fa-kit`. There are even (Jupyter notebooks)[https://github.com/bmcmenamin/fa_kit/tree/master/examples] that walk you through how to use it so I don't need to write a ton of documentation here.

Have fun, and remember -- if you're having trouble making sense of your multivariate dataset, just say "fa-kit".
