---
layout: post
title: "Musings on high-dimensional fluff-balls"
author: "Brenton McMenamin"
---

I read [this blog post](http://www.inference.vc/high-dimensional-gaussian-distributions-are-soap-bubble/) about normal distributions before the holidays, and it's been stuck in my head ever since.

Go read the whole, thing, but here's the thing I want to focus on:

> If I show you a Gaussian distribution in 3D and ask you to find a physical metaphor for it, I think at least some of you would reasonably describe it as something like a ball of mold: it's kind of soft or fluffy but gets more and more dense in the middle. ... But in high-dimensional spaces, the image you should have in your mind is more like a soap bubble

This is a really counter-intuitive idea. Anybody that was half-awake during an intro to stats course could describe what a Gaussian distribution looks like -- there's a clear 'central' point where observations are relatively common, and as you get further away from that central point observations become less and less likely. This description works well for 1- and 2- dimensional Gaussians, so it makes sense to keep generalizing it to higher dimensions. But how do you *know* that this trend holds and 3+ dimensional Gaussians have the same properties? The simple test is to make a bunch of graphs that let us see whether these properties hold up. For example, we could draw a bunch of samples from a 3-d Gaussian, and then project the results to scatterplots of the XY and YZ planes. Unfortunately, doing this still ends with us making judgements about a series of low-dimensional shapes and trying to infer the higher-dimensional structure.

A more rigorous approach is to calculate the expeted distance between a sample of the distribution and the distribution mean. If the distribution has a fluff-ball shape, we'd expect this distribution to peak at 0 and decrease as distance increases; if the distribution has a soap-bubble shape, we'd expect there to be some peak at a non-zero value where the 'wall' of the bubble exists. Luckily, this is really easy to do for a $k$-dimentional sphereical Gaussians centered at $0$. The squared-euclidean distance between a sample and the center of the distribution is $\mathcal{D} = \sum_{i=1}^k {\mathcal{N}_i(0, 1)}^{2}$. The sum of squared Gaussians is distributed as a $\mathcal{\chi^2}$-distrubtion with [$k$ degrees of freedom](https://en.wikipedia.org/wiki/Chi-squared_distribution#Definition), so $\mathcal{D} \sim \mathcal{\chi^2}(k)$.

Plotting the $\mathcal{\chi^2}(k)$ $pdf$ will show us how points are distributed around the origin for our multivariate Gaussian. Thankfully, wikipedia has been kind enough to plot out this $pdf$ with several different degrees of freedom, and we see that there's a big difference between $pdf$s with 1 or 2 degrees of freedom and those with 3 or more:

<div style="text-align: center;">
    <a title="By Geek3 (Own work) [GFDL (http://www.gnu.org/copyleft/fdl.html) or CC BY 3.0 (http://creativecommons.org/licenses/by/3.0)], via Wikimedia Commons" href="https://commons.wikimedia.org/wiki/File%3AChi-square_pdf.svg">
        <img width="512" alt="Chi-square pdf" src="https://upload.wikimedia.org/wikipedia/commons/thumb/3/35/Chi-square_pdf.svg/512px-Chi-square_pdf.svg.png"/>
    </a>
</div>

The distributions for 1 or 2 degrees of freedom peak at 0 and decrease, whereas those with 3 or more degrees of freedom have a peak at some non-zero value.

That means that most of the samples from a 1 or 2 dimensional Gaussian will amass around the origin and become less common as you move away, explainign why our visualizations of a 1 or 2 dimensional Gaussian give us the impress that it's a fluff-ball. However, higer dimensional Gaussians will have the most samples amass at a fixed distance away from the origin, corresponding to a soap-bubble shape.


## So do fluff-ball distributions exist at all?

So, the Gaussian is most definitely *not* a fluff-ball in high-dimensions. But does such a distribution actually exist?

### You can't build a fluff-ball distribution by concatenating a bunch of $i.i.d.$ univariate distributions

Given that a spherical multivariate Gaussian can be made by stacking together a whole bunch of independent univariate Gaussians, a natural approach to building a $k$-dimensional fluff-ball would be to pick out some other candidate univariate distribution that's centered as 0 (*e.g.* replacing the Gaussian with a Laplacian or Cauchy distribution), $\mathcal{F}$, and just stack together $k$ samples from it to make a multivariate as $[\mathcal{F}_1, \mathcal{F}_2, ..., \mathcal{F}_k]$. It turns out that this is guaranteed to not work for *any* candidate $\mathcal{F}$.

We can write the distribution of expected distances between a sample and the origin as $\mathcal{D_F} = \sum_{i=1}^k {\mathcal{F}_i^2}$. Previously, when $\mathcal{F}$ was Gaussian, we knew that $\mathcal{D_F}$ followed a $\mathcal{\chi^2}$ distribution. We can't determine the exact shape of the dsitribution in the general case, but thanks to the [Central Limit Theorem (CLT)](https://en.wikipedia.org/wiki/Central_limit_theorem) we know that $\mathcal{D_F}$ will converge to a Gaussian distribution as $k \to \inf$ because it is defined as the summation of $i.i.d.$ random variables. And that means that $\mathcal{D_F}$ will *not* have the shape corresponding to a fluff-ball distribution (i.e., a peak at zero and constantly decreasing probability).

### ... but they *do* exist.

The CLT proves that we can't build a fluff-ball using independent univariate distributions, but that still leaves open the possibiltiy that we can build such a distribution if we allow statistical dependence between dimesnions. Intuitively, this makes sense -- if we want to ensure that most values of the distribution occur near the origin, we need a way to say "we've drawn some extreme values in the first $i$ dimensions, so we should be extra carfeul not to draw anything too big in the remainin $k-i$ dimensions".

So, how do we build this distribution? Change to a [spherical coordinate system](https://en.wikipedia.org/wiki/N-sphere#Spherical_coordinates) that defines a point in $k$-dimensional space using a radius $r$ and $k-1$ azimuth angles $\phi_i$, such that $r > 0$, $\phi_1 \in [0, 2\pi)$, and $\phi_i \in [0, \pi)$ for $i > 1$.

In this coordinate space, it's almost trivial to define a fluff-ball distribution. First, we want the distribution to be 'rotationally invariant' and not have a preferred direction in space, so we set the distriution of each azimuth to uniform across all their allow angles:

$$
\begin{eqnarray} 
\phi_1 &\sim& U(-\pi, \pi) \nonumber \\
\phi_2 &\sim& U(0, \pi) \nonumber \\
&...& \nonumber \\
\phi_{k-1} &\sim& U(0, \pi) \nonumber \\
\end{eqnarray} 
$$

Second, we need to put a distribution on $r$ that will give us the desired distribution of distances from the origin. For a Gaussian, we know that the distribution of squared euclidean distances follow a $\chi^2(k)$-distibution, which is equivalent to the [gamma distribution](https://en.wikipedia.org/wiki/Gamma_distribution), $\Gamma(\frac{k}{2}, \frac{1}{2})$. However, for a fluff-ball distribution, we'd like something shaped like an [Exponetial distribution](https://en.wikipedia.org/wiki/Exponential_distribution), $Exp(\lambda)$, which is equivalent to $\Gamma(1, \frac{1}{\lambda})$. So we could recreate the Gaussian bubble-like distribution by ensuring the $r^2 \sim \Gamma(\frac{k}{2}, \frac{1}{2})$, or we could create a fluff-ball distribution by changing the gamma distribution parameteres so that r^2 \sim \Gamma(1, \frac{1}{\lambda})$.

### Now comes the hard part

We were able to parametrize a fluff-ball distribution with simple probailitsic distributions in sphereical-coordinates. But what happens if we project this back into the original Cartesian coordiante system?


### The 2-dimensional example

Once we project the 2-dimensional example from spherical ($r$, $\phi_1$) to Cartesion ($x_1$, $x_2$) coordinates, the statistical dependance across dimensions is obvious:

$$
\begin{eqnarray} 
x_1 &=& r\!\cdot\!\sin(\phi_1) \nonumber \\
x_2 &=& r\!\cdot\!\cos(\phi_1) \nonumber \\
\end{eqnarray} 
$$

Moveover, because $\phi_1 \sim U(-\pi, \pi)$ covers the whole a period of both $sin(\phi_1)$ and $cos(\phi_1)$, we know that the form of the $pdf$s for $sin(\phi_1)$ and $cos(\phi_1)$ will be the same. That means that the marginal distributions of $x_1$ and $x_2$ will be identical same.

But what *is* the form of that marginal distribution?

Based on [stackoverflow](https://math.stackexchange.com/a/1026376) and [other blogs](http://atif-razzaq.blogspot.com/2011/02/probability-density-function-pdf-of.html), we know that $sin(\phi_1)$ will take on values in $x \in (-1, 1)$ with probability $f(x) = \frac{1}{\pi \sqrt{1 - x^2}} = \frac{1}{\pi \sqrt{(x + 1)(1 - x)}}$ which is equivalent to the [$Arcsine(-1, 1)$ distribution](https://en.wikipedia.org/wiki/Arcsine_distribution).


And assuming that the distribution for $r$ follows some version of the $\sqrt{Gamma(\alpha, \beta)}$ distribution, the entire pdf for either $x_i$ is:










# Nothing below here makes sense... yet




$$
\begin{eqnarray} 

f_{\alpha, \beta}(x) &=& \left (
                            \sqrt{\frac{\beta^\alpha}{\Gamma(\alpha)} x^{\alpha - 1} e^{-\beta x}}
                        \right )
                        \left (
                            \frac{1}{\pi \sqrt{1 - x^2}}
                        \right )
                    \nonumber \\

\end{eqnarray}
$$

For a bubble-like distribution, we set $r \sim $\sqrt{Gamma(\frac{k}{2}, \frac{1}{2})} = $\sqrt{Gamma(1, \frac{1}{2})}$, which simplifies the full $pdf$ to:


3

$$
\begin{eqnarray} 

f_{1, \frac{1}{2}}(x) &=& \left (
                                \sqrt{\frac{\frac{1}{2}}{\Gamma(1)} x^{0} e^{-\frac{1}{2} x}}
                            \right )
                            \left (
                                \frac{1}{\pi \sqrt{1 - x^2}}
                            \right )
                    \nonumber \\

                      &=& \left (
                                \sqrt{\frac{1}{2} e^{\frac{-x}{2}}}
                            \right )
                            \left ( 
                                \frac{1}{\pi \sqrt{1 - x^2}}
                            \right )
                    \nonumber \\

                      &=& \frac{1}{ \pi \sqrt{2} }
                          \frac{ e^{\frac{-x}{4}} }{ \sqrt{1 - x^2} }
                    \nonumber \\

\end{eqnarray}
$$







### The 3-dimensional example


$$
\begin{eqnarray} 
x_1 &=& r\!\cdot\!\sin(\phi_1)\!\cdot\!\sin(\phi_2) \nonumber \\
x_2 &=& r\!\cdot\!\cos(\phi_1)\!\cdot\!\sin(\phi_2) \nonumber \\
x_3 &=& r\!\cdot\!\cos(\phi_2) \nonumber \\
\end{eqnarray}
$$


Now we also need to know how $\sin(\phi_2)$ and $\cos(\phi_2)$ are distributions, 
Given that $\phi_2 \sim U(0, \pi)$ rather than the full period of $U(-\pi, \pi)$ for $\phi_1$, we will now notice a difference in the distribution of $sin$ and $cos$.

The distribution of $cos(\phi_2)$ will still be $Arcsine(-1, 1)$, but 

$\sin(\phi_2)$ is going to be restricted to the positive half of the distribution. We can get at it's form by combining this trig identitiy:

$$
\begin{eqnarray}
\cos(\alpha)^2 + \sin(\alpha)^2 &=& 1  \nonumber \\
\sin(\alpha)^2 &=& 1 - \cos(\alpha)^2  \nonumber \\
\sin(\alpha) &=& \sqrt{1 - \cos(\alpha)^2}  \nonumber \\
\end{eqnarray}
$$

With the fact that $\cos(\alpha) \sim \frac{1}{\pi \sqrt{1 - x^2}}$, and we find that:

$$
\begin{eqnarray}
\sin(\alpha) &=& \sqrt{1 - \cos(\alpha)^2}  \nonumber \\
             &=& \sqrt{1 - (\frac{1}{\pi \sqrt{1 - x^2}})^2}  \nonumber \\
             &=& \sqrt{1 - \frac{1}{\pi^2 (1 - x^2)^2}}  \nonumber \\
             &=& \sqrt{\frac{\pi^2 (1 - x^2)^2 - 1}{\pi^2 (1 - x^2)^2}}  \nonumber \\
             &=& \frac{\sqrt{\pi^2 (1 - x^2)^2 - 1}}{\pi (1 - x^2)}  \nonumber \\
             &=& \frac{\sqrt{(\pi (1 - x^2) + 1)(\pi (1 - x^2) - 1)}{\pi (1 - x^2)}  \nonumber \\
\end{eqnarray} 
$$



### The >3-dimensional example

No. I'm not doing this. I write this stuff for *fun*.

