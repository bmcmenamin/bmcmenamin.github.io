---
layout: post
title: "Learning receptive fields with Hebbian networks"
author: "Brenton McMenamin"
tags: [hebbian-networks]
---

I recently started a repo to explore [Hebbian/Anti-hebbian networks]({{ site.baseurl }}{% post_url 2018-10-27-hebb %}) but was having issues replicating some of Pehlevan & Chklovskii's results.

Thanks to some help from the original authors, I've been able to track down a number of issues in my implementation and have been able to make some notebooks that use my models to replicate their findings that these networks can [extract the top N principle components](https://github.com/bmcmenamin/hebbnets/blob/master/demos/Test%20extraction%20of%20top%20principal%20components.ipynb) from an image or be modified to learn image representations that [appear similar to receptive fields from V1](https://github.com/bmcmenamin/hebbnets/blob/master/demos/Test%20learning%20receptive%20fields%20from%20natural%20from%20images.ipynb).


See? Receptive-field-ish blobs that have localized orientated patterns!

<div align="center">
    <img alt="I guess these are like receptive fields?" src="/figs/hebb_rf/learned_v1_rfs.png" width="600px">
</div>
