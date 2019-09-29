---
layout: post
title: "Rotation Invariances in Different Classifiers"
author: "Brenton McMenamin"
tags: [math&data]
---


Data scientists frequently apply ML models to multi-dimensional inputs where the coordinate systems are arbitrary, such as data that's been embedded in a high-dimensional space (_e.g._, Word2Vec) or run through a dimensionality-reduction step as part of pre-processing (_e.g._, pre-transforming data with PCA or UMAP). When doing this, it's important to consider whether the model you're using cares about the coordinate system of the inputs.

**Rotationally invariant** models don't care about which coordinate system you use, so they'd perform equally well on two tasks where the same data has been rotated to a different coordinate system. For example, classifying using input features (X, Y) would perform the same a classifying on a "rotated" set of features that like (X + Y, X - Y). But some commonly-used simple models, as we'll see below', would perform differently on these two rotations.

I've whipped up some code and demo notebooks [here](https://github.com/bmcmenamin/sundries/blob/master/rotation_invariant_classifiers/RotationDemo.ipynb) that'll let you test how arbitary Scikit-Learn models handle coordinate rotations. The demo will create two clouds of points in a two-dimensional space, then train and test your classifier on that same point cloud under a variety of rotatations. This will let you visualize how gradual changes to the coordinate system affect the classifier's performance.


## Basic Logistic Classification

To start, let's look at how standard linear logistic classifier handles rotation.

The cloud of points is in the upper left panel below. Half of them are used to train the classifier, and the other half are used to test and generate the ROC curve (lower left) and track the overall model performace over the course of rotation using the AUC (lower right). Notice that the ROC curve and AUC values are constant as the points rotate -- that's because the linear logistic classifier is rotationally invariante and doesn't care about the input point coordinate system.

<br>
<div align="center">
    <video alt="Logistic" width="600px" autoplay loop>
    <source src="/figs/rotation_invariance/Logistic_Rotation.mp4" width="600px">
    </video>
</div>
<br>

The upper left provides a visualization of the model's decision boundary, so we can see that the model is using a simple linear bounary to separate the datapoints and that it is changing smoothly and in-synch with the rotations.

## Logistic Classification, with L1 Regularization

But what happens when we start applying L1 regularization to linear model? (Side-note: always read the docs so you know what the [default parameters are doing!](https://ryxcommar.com/2019/08/30/scikit-learns-defaults-are-wrong/).

Using L1 regularization changes model training so there is always a tradeoff between making the model better at classification and making the model "simpler" (i.e., the model prefers to have it's parameters near zero). So now when the points rotate, we see an interesting effect on the decision boundary:

<br>
<div align="center">
    <video alt="Logistic L1" width="600px" autoplay loop>
    <source src="/figs/rotation_invariance/LogisticL1_Rotation.mp4" type="video/mp4">
    </video>
</div>
<br>

Whenever the decision boundary becomes horizontal or vertical, it stops rotating for a moment "sticks" there.

That's because the horizontal (or vertical) decision boundaries are "simple" in the sense that they only require using a single input dimension (i.e., the x or y axis), so the regulariation will push the model to use these boundaries even if they're not optimal for classification. You can see this tradeoff happening in the lower right as the model decides to build a classifier with lower AUC when favoring the "simple" horizontal and vertical dimensions.


## Decision Trees

Finally, let's look at how a Decision Tree handles rotation.

The previous linear models can easily create decision boundaries that combine all the input features to make a diagonal boundary. For example, if the input data were rotated 45 degrees, a linear model

could come up with `if x - y > 0 then A else B` and do really well. But a Decision Tree can't do that -- it operates on each input feature sequentially, so it would have to approximate that with something gross like `if x > 0 AND x <= 4 AND y > 2 then MOST LIKEY A else B`. We can see these nasty rectilinear approximations to a smooth diagonal decision boundary (and their affect on overall model performance) here:

<br>
<div align="center">
    <video alt="Decition Tree" width="600px" autoplay loop>
    <source src="/figs/rotation_invariance/DecisionTree_Rotation.mp4" type="video/mp4">
    </video>
</div>
<br>