---
layout: post
title: "title about soap?"
author: "Brenton McMenamin"
---

I read [this blog post](http://www.inference.vc/high-dimensional-gaussian-distributions-are-soap-bubble/) about normal distributions before the holidays, and it's been stuck in my head ever since.

Go read the whole, thing, but here's the thing I want to focus on:

> If I show you a Gaussian distribution in 3D and ask you to find a physical metaphor for it, I think at least some of you would reasonably describe it as something like a ball of mold: it's kind of soft or fluffy but gets more and more dense in the middle. ... But in high-dimensional spaces, the image you should have in your mind is more like a soap bubble

This is a really counter-intuitive idea. Anybody that was half-awake during an intro to stats course could describe what a Gaussian distribution looks like -- there's a clear 'central' point where observations are relatively common, and as you get further away from that central point observations become less and less likely. This description works well for 1- and 2- dimensional Gaussians, so it makes sense to keep generalizing it to higher dimensions. But how do you *know* that this trend holds and 3+ dimensional Gaussians have the same properties? The simple test is to make a bunch of graphs that let us see whether these properties hold up. For example, we could draw a bunch of samples from a 3-d Gaussian, and then project the results to scatterplots of the XY and YZ planes. Unfortunately, doing this still ends with us making judgements about a series of low-dimensional shapes and trying to infer the higher-dimensional structure.

A more rigorous approach is to calculate the expeted distance between a sample of the distribution and the distribution mean. If the distribution has a fuzz-ball shape, we'd expect this distribution to peak at 0 and decrease as distance increases; if the distribution has a soap-bubble shape, we'd expect there to be some peak at a non-zero value where the 'wall' of the bubble exists. Luckily, this is really easy to do for a $d$-dimentional sphereical Gaussians centered at $0$. The squared-euclidean distance between a sample and the center of the distribution is $\sum_{i=1}^d {\mathcal{N}(0, 1)}^{2}$, which corresponds to a $\mathcal{\chi^2}$-distrubtion with [$d$ degrees of freedom](https://en.wikipedia.org/wiki/Chi-squared_distribution#Definition).

That means we can plot the $\mathcal{\chi^2}(d)$ p.d.f. to see how points are distributed in the multivariate Gaussian relative to the centroid. Wikipedia has been kind enough to plot out this p.d.f for several different degrees of freedom for us, and we see that there's a big different when going from 2 to 3 degrees of freedom:

<a title="By Geek3 (Own work) [GFDL (http://www.gnu.org/copyleft/fdl.html) or CC BY 3.0 (http://creativecommons.org/licenses/by/3.0)], via Wikimedia Commons" href="https://commons.wikimedia.org/wiki/File%3AChi-square_pdf.svg"><img width="512" alt="Chi-square pdf" src="https://upload.wikimedia.org/wikipedia/commons/thumb/3/35/Chi-square_pdf.svg/512px-Chi-square_pdf.svg.png"/></a>

The distributions for 1 or 2 degrees of freedom peak at 0 and decrease -- meaning that most points would be near the center and the distribution would look like a fuzz-ball. But starting with dimension 3, there's a non-zero peak for the distance distribution that would correspond to the distribution taking on a bubble-like shape. So, this means that all of our low-dimensional views of high-dimensional Gaussians are going to obscure it's true bubble-y shape.


## So do fuzz-ball distributions exist at all?


Can we find *any* form of probability distribution that would give us a high-dimensional fuzz ball?


### You can't make a fuzz ball out of *any* massively univariate distribution

The spherical multivariate Gaussian is made be stacking together a whole bunch of independant univariate Gaussians. Is there some *other* univariate distribution, $\mathcal{F}$, that we could stack into a fuzz-ball multivariate?

We know that the distribution of expected distances between a point in this distribution and it's center would be $\sum_{i=1}^d {\mathcal{F}^2}$, and we'd need that distribution to have a peak at zero and be constantly decreasing. However, the Central Limit Theorem tells us that this distribution would start to approximate a Gaussian when we get into high dimensions, meaning that the distribution would take on a bubble-like shape rather than a fuzz-ball shape.

### ... but you can build it.

let's build it from scratch

polar coordinates:

r distributed with something in the range $(0, \inf)$, decreasing
all others are $[0, 2*pi)$ to ensure rotational invariance


But what would the marginal distribution look like?

as a formula (showing non-independance)
as a plot