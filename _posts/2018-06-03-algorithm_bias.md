---
layout: post
title: "Algorithmic solutions for algorithmic biases?"
author: "Brenton McMenamin"
tags: [math&data, model-wrangler]
---

If you're building machine learning algorithms that affect anything in the real world, you should be familiar with the consequences of machine bias. If you're not, [this](https://www.propublica.org/series/machine-bias) is a good place to start. Researchers are developing modifications to existing algorithms to encourage machines to learn fair, unbiased models. But, there's considerable difficulty due to the difficulty of defining *fair*, the fact that different definitions of fair are incompatible. See [this](https://arxiv.org/pdf/1609.05807.pdf) and [this](https://arxiv.org/pdf/1701.08230.pdf) and [this](https://www.propublica.org/article/bias-in-criminal-risk-scores-is-mathematically-inevitable-researchers-say)

To that end, I've started playing with this concept on my own by building new loss functions and model architectures in [`model_wrangler`](https://github.com/bmcmenamin/model_wrangler) that will allow me to add 'fairness' to the optimization functions used by model wrangler. In the latest PR has added:

* The `loss_crossgroup_bias` model loss function.
  * This loss takes three arguments -- the current model predictions, the actual values, and a categorical index that indicates the 'group' of each sample. The loss function calculates the prediction error for each observation (*i.e.*, observed - actual), averages this error within each group to measure per-group bias, and returns the variance of this score across groups. Adding this loss to the overall model loss function will penalize the model if some groups have predictions that are systematically skewed relative to the other groups.
  * Note: in practice, adding this loss function will require a larger batch size (to there are samples from each groups) and will require more training epochs to converge.

* The `DebiasedClassifier` model class in the corral creates a dense feedforward classifier that adds the `loss_crossgroup_bias` penalty to its optimization function. The 'group' labels for each sample are provided as 0-indexed integers and feed in as the last input.

I expect more updates and features once I solve this unsolvable problem.

