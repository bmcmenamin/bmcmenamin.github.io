---
layout: post
title: "Musings on high-dimensional fluff-balls"
author: "Brenton McMenamin"
tags: [math]
---

I read [this blog post](http://www.inference.vc/high-dimensional-gaussian-distributions-are-soap-bubble/) about normal distributions before the holidays, and it's been stuck in my head ever since.

Go read the whole, thing, but here's the thing I want to focus on:

> If I show you a Gaussian distribution in 3D and ask you to find a physical metaphor for it, I think at least some of you would reasonably describe it as something like a ball of mold: it's kind of soft or fluffy but gets more and more dense in the middle. ... But in high-dimensional spaces, the image you should have in your mind is more like a soap bubble

This is a really counter-intuitive idea. Anybody that was half-awake during an intro to stats course could describe what a Gaussian distribution looks like -- there's a clear 'central' point where observations are relatively common, and as you get further away from that central point observations become less and less likely. This description works well for 1- and 2- dimensional Gaussians, so it makes sense to keep generalizing it to higher dimensions. But how do you *know* that this trend holds and 3+ dimensional Gaussians have the same properties? The simple test is to make a bunch of graphs that let us see whether these properties hold up. For example, we could draw a bunch of samples from a 3-d Gaussian, and then project the results to scatterplots of the XY and YZ planes. Unfortunately, doing this still ends with us making judgements about a series of low-dimensional shapes and trying to infer the higher-dimensional structure.

A more rigorous approach is to calculate the expected distance between a sample of the distribution and the distribution mean. If the distribution has a fluff-ball shape, we'd expect this distribution to peak at 0 and decrease as distance increases; if the distribution has a soap-bubble shape, we'd expect there to be some peak at a non-zero value where the 'wall' of the bubble exists. Luckily, this is really easy to do for a $k$-dimensional spherical Gaussians centered at $0$. The squared-euclidean distance between a sample and the center of the distribution is $\mathcal{D} = \sum_{i=1}^k {\mathcal{N}_i(0, 1)}^{2}$. The sum of squared Gaussians is distributed as a $\mathcal{\chi^2}$-distrubtion with [$k$ degrees of freedom](https://en.wikipedia.org/wiki/Chi-squared_distribution#Definition), so $\mathcal{D} \sim \mathcal{\chi^2}(k)$.

Plotting the $\mathcal{\chi^2}(k)$ $pdf$ will show us how points are distributed around the origin for our multivariate Gaussian. Thankfully, wikipedia has been kind enough to plot out this $pdf$ with several different degrees of freedom, and we see that there's a big difference between $pdf$s with 1 or 2 degrees of freedom and those with 3 or more:

<div style="text-align: center;">
    <a title="By Geek3 (Own work) [GFDL (http://www.gnu.org/copyleft/fdl.html) or CC BY 3.0 (http://creativecommons.org/licenses/by/3.0)], via Wikimedia Commons" href="https://en.wikipedia.org/wiki/Chi-squared_distribution">
        <img width="512" alt="Chi-square pdf" src="https://upload.wikimedia.org/wikipedia/commons/thumb/3/35/Chi-square_pdf.svg/512px-Chi-square_pdf.svg.png"/>
    </a>
</div>

The distributions for 1 or 2 degrees of freedom peak at 0 and decrease, whereas those with 3 or more degrees of freedom have a peak at some non-zero value. (Fun fact: a $\mathcal{\chi^2}(2)$ distribution is the *same* as an exponential distribution. That'll be important later.)

That means for a 1 or 2 dimensional Gaussian, most of the samples will amass near the origin and become less and less frequent as you move away, explaining why visualizations of a 1 or 2 dimensional Gaussian look like a fluff-ball. However, higher dimensional Gaussians have their modal samples at a fixed distance away from the origin, with samples being rarer on either side of the boundary. Unfortunately, this soap-bubble structure *only* appears in dimension 3 or higher so it's hard to visualize.

## So do fluff-ball distributions exist at all?

So, the Gaussian is most definitely *not* a fluff-ball in high-dimensions. But does such a distribution actually exist?

### You can't build a fluff-ball distribution by concatenating a bunch of $i.i.d.$ univariate distributions

Given that a spherical multivariate Gaussian can be made by stacking together a whole bunch of independent univariate Gaussians, a natural approach to building a $k$-dimensional fluff-ball would be to pick out some other candidate univariate distribution that's centered as 0 (*e.g.* replacing the Gaussian with a Laplacian or Cauchy distribution), $\mathcal{F}$, and just stack together $k$ samples from it to make a multivariate as $[\mathcal{F}_1, \mathcal{F}_2, ..., \mathcal{F}_k]$. It turns out that this is guaranteed to not work for *any* candidate $\mathcal{F}$.

We can write the distribution of expected distances between a sample and the origin as $\mathcal{D_F} = \sum_{i=1}^k {\mathcal{F}_i^2}$. Previously, when $\mathcal{F}$ was Gaussian, we knew that $\mathcal{D_F}$ followed a $\mathcal{\chi^2}$ distribution. We can't determine the exact shape of the dsitribution in the general case, but thanks to the [Central Limit Theorem (CLT)](https://en.wikipedia.org/wiki/Central_limit_theorem) we know that $\mathcal{D_F}$ will converge to a Gaussian distribution as $k \to \inf$ because it is defined as the summation of $i.i.d.$ random variables. And that means that $\mathcal{D_F}$ will *not* have the shape corresponding to a fluff-ball distribution (i.e., a peak at zero and constantly decreasing probability).

### ... but they *do* exist.

The CLT proves that we can't build a fluff-ball using independent univariate distributions, but that still leaves open the possibility that we can build such a distribution if we allow statistical dependence between dimensions. Intuitively, this makes sense -- if we want to ensure that most values of the distribution occur near the origin, we need a way to say "we've drawn some extreme values in the first $i$ dimensions, so we should be extra careful not to draw any samples with extreme marginals for the other $k-i$ dimensions".

So, how would we build this distribution? Changing to a [(hyper)spherical coordinate system](https://en.wikipedia.org/wiki/N-sphere#Spherical_coordinates) is an helpful first step. This coordinate systems defines points in a $k$-dimensional space using a radius $r$ and $k-1$ azimuth/elevation angles $\phi_i$, such that $r > 0$, $\phi_1 \in [0, 2\pi)$, and $\phi_i \in [0, \pi)$ for $i > 1$.

First, we can put a distribution on $r$ that will ensure that the distribution is appropriately fluffy or soapy by controlling the distribution of distances from the origin.

For a soap-bubble, we know that the Gaussian distribution has squared euclidean distances that follow a $\chi^2(k)$-distribution, which is equivalent to the [gamma distribution](https://en.wikipedia.org/wiki/Gamma_distribution), $\Gamma(\frac{k}{2}, \frac{1}{2})$, so we can use that. For the fluff-ball distribution, we'd like something that's always decreasing just like the $\chi^2$ distro did at lower degrees of freedom. A natural fit here is the [Exponetial distribution](https://en.wikipedia.org/wiki/Exponential_distribution), $Exp(\lambda)$, defined as $\Gamma(1, \frac{1}{\lambda})$, which can be made *identical* to the $\chi^2$ with lower degrees of freedom.

Up next, we just need to define the distributions on the $n-1$ angular coordinates to allow the distribution to be rotationally invariant. However, [actually doings this](http://mathworld.wolfram.com/HyperspherePointPicking.html) is not nearly as simple as you'd expect. Or, maybe you *would* expect it and I was just naive going in to this. I'm not going to work it out here, but just trust me that there's a way to play arcsine distributions or whatever on the remaining $k-1$ angular variables to ensure that all 'directions' are equally likely.

### What do the marginals look like?

I originally wanted to come up with an analytical closed-form description of what the marginal distributions would look like for the fluff-ball if we projected it back to Cartesian coordinates. But it turned out to be pretty messy and I didn't want to spend forever writing out MathJax, so I decided to use the inelegant brute-force method of generating histograms with random sampling. I didn't even need to pull out any fancy probabilistic programming languages!

Here's a plot showing the marginal distributions of values for data distributed as a soap-bubble (blue), fluff-ball (red), or multivariate Gaussian (black) and the dimensionality of the sphere increases. (Notebook for making the plot lives here)[https://github.com/bmcmenamin/sundries/blob/master/fluff_ball/Plotting%20a%20fluffball.ipynb]

<div style="text-align: center;">
    <a title="Marginal distributions of soap-bubble (blue), fluff-ball (red), or multivatiate Gaussian (black) and the dimensionality of the sphere increases" href="https://github.com/bmcmenamin/sundries/blob/master/fluff_ball/Plotting%20a%20fluffball.ipynb">
        <img width="512" alt="Marginal distributions of spheres" src="/figs/spheres/marginal_distros.png"/>
    </a>
</div>

In the top panel, for the 2-dimensional sphere (what *some* people would call a circle), we see that all three distributions are identical. However, as we move into the higher dimensional spaces, we see that the fluff-ball deviates from the the soapy/Gaussian distributions by over-emphasizing values near zero. I've got a hunch that it's actually distribution as a [Cauchy distribution](https://en.wikipedia.org/wiki/Cauchy_distribution), but I'm not in a rush to prove it.


# Conclusions

* High dimensional Gaussians don't look the way you think they look. The 'fluff-ball' type distribution you're probably imagining a) cannot be built from independent univariate distributions and b) has a sharper (more kurtotic?) distribution than a normal.

* Also, it's easy to underestimate the difficulty of drawing random sampling on an $n$-sphere using hyperspherical coordinates.
