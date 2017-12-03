---
layout: post
title: "Adding Siamese Networks for learning embeddings to the ModelWrangler"
author: "Brenton McMenamin"
---


I've added a first round of support to ModelWrangler for siamese networks, so you can learn embeddings for any type of input.

I run it over a subset of the MNIST in the example file so it creates an embedding space where images of numbers will be embedded next to other images of the same digit. The example is pretty watered down becaue I wanted to run it on a CPU in a reasonable amount of time, but the output looks pretty good!

When that embedding space is rendered in two-dimensions using TSNE, you see this:

<img src="/figs/model_wrangler/mnist_siamese_example.png" width="600">

Digits are mostly grouped together by their same type of digits, and similar looking groups of digits (0 and 8, 1 and 7, etc) end up next to each other.

