---
layout: post
title: "Grid search with the 'Reusable holdout'"
author: "Brenton McMenamin"
---
 
One of the cardinal rules of data-science/machine-learning is to use separate datasets for testing and training your models. This is because models will _overfit_ the data they are trained on, so if you test a model using the data that went into it's own training the performance will be grossly inflated and you'll think you're doing much better than you actually are.
 
This means that modelling *should* use the following workflow:
 
1. Train a model on the training set
2. Test the model on the test set and see if it did well
 
In reality it rarely ends up that way and a third step is often added that  introduces a feedback loop:
 
3. Notice something peculiar about how the model performed during the test, and decided to retrain the model in a different way. For example, "this model isn't is doing something strange with the data from feature _i_ during the test... let's just retrain it without that feature to see what happens."
 
The problem with this new step is that it allows things that you observed in the testing data to guide the development of the model. This means that the model is explicitly overfit to the training data (from the actual model fitting/training step) and implicitly overfit to the test data (because this iterative development process causes you to select models that maximize performance on the test data).
 
To get around this feedback problem, modellers often separate their data into three bins -- training, test, validation -- rather than just the training and test. That way you can build all the variants of your model (*e.g.*, doing a grid search over various parameters), train every model on training data, calculate their performance on the test data, and then select the best model. Then the best model is tested on the validation data to get an 'unbiased' measure of how well it does. Conceptually, this is works but now you're partitioning your data into more, smaller groups that are each getting overfit by a different mechanism. You could keep diving your dataset up like this, but eventually you'll run out of data.
 
Fixing this problem is why the data-world was abuzz with Dwork *et al*'s "reusable holdout" a couple years ago -- their paper proposed a probabilistic method named `thresholdOut` that limits how a user can access the data they've held out for a test group; by carefully injecting randomness into the held-out dataset, you can keep re-using the dataset without overfitting to it. This paper was so exciting that it actually got published in a [real journal](http://science.sciencemag.org/content/349/6248/636) instead of just getting passed around on arXiv indefinitely. It's a short article that you should read, but it's paywalled. So here another good [write-up](https://research.googleblog.com/2015/08/the-reusable-holdout-preserving.html).
 
### The original Dwork demonstration
 
I was intrigued by the promise of a reusable holdout dataset, so I worked through a python version of the example in Dwork's paper [here](https://github.com/bmcmenamin/thresholdOut-explorations/blob/master/original_method/Threshold%20out%20demos%20--%20Dwork's%20original%20method.ipynb).
 
In their example, an ensemble of nearest neighbors classifier was trained in a way that ensured overfitting to the held-out test set (both the training and 'holdout' sets were used for the model's feature selection). When using standard methods, the model overfit the training data and the hold-out dataset relative to a completely fresh set of data used for comparison. However, when access to the hold-out dataset was only allowed via the `thresholdOut` function, the model did not show overfitting on the hold-out dataset. 
 
This experimental setup aws an elegant demonstration of their algorithm and had two big advantages for the paper:
1. You could easily adjust the strength of the overfitting parametrically to demonstrate the robustness of the reusable holdout
2. The simplicity of the model made it clear that the success of thresholdout wasn't  some sleight-of-hand that occurred during a complex model-training/fitting (e.g., that you're doing something numerically that affects model convergence in one condition rather than prevent overfitting *per se*)
 
But their experimental setup doesn't really resemble the way I work with supervised learning algorithms or how I would plausibly use `thresholdOut` in the wild. So I decided to put together another test and see how well `thresholdOut` would do at preventing overfitting in a grid-search for tuning model hyperparameters.
 
 
#### Threshold out on grid search for a Logistic Regression model
 
 
I set up a test of `thresholdOut` that would work in more conventional grid-search format in [this notebook](https://github.com/bmcmenamin/thresholdOut-explorations/blob/master/Threshold%20out%20demos%20--%20tuning%20parameters%20for%20linear%20regression.ipynb). Each test starts by generating three random datasets -- training, holdout, and test. Then a grid-search was used to train a whole bunch of L1-regularized logistic regression models with different regularization parameters. Each model was trained on the training data and evaluated using the holdout data.
 
* For a 'standard' analysis, I picked the model with the highest performance on the holdout data set and compared the observed accuracy their to it's accuracy on the 'test' dataset.
 
* For the 'thresholdOut' analysis, I picked the model with the highest `thresholdOut` score (calculated from its accuracy on training and holdout) and compared _that_ model's holdout accuracy to 'test' accuracy.
 
This was repeated 50 times on different random datasets with a known relationship for the classifier to learn, and another 50 times on datasets without an explicit relationship where we'd except chance (50%) accuracy.
 
##### Results
 
The top two panels show model accuracies for datasets where there *was* a relationship to learn, and the bottom two panels show the results for when there was no known relationship.
 
 
The models overfit the training data in all cases (*i.e.*, the blue bars are far above the red bars).
 
The two left panels show are where the 'standard' grid search method was used, and they show that there's overfitting in the holdout set (*i.e.*, the green bars are above the red bars). The amount of overfitting is relatively smaller in the two right panels which depict the `thresholdOut` grid search method. Overall, this means that if you use `thresholdOut` to pick the best model in your grid search you may not need to have another clean test dataset around to test the overall model performance.
 
 
<img src="/figs/dwork/thresholdOut_classifier_results.png" width="600">
 
### Conclusions and questions
 
The main conclusion is that `thresholdOut` works in a conventional supervised learning pipeline where you need to optimize parameters over a grid-search. However, I still have some lingering questions that I'll need to resolve before I put something like this into my standard bag of tricks. For example:
  
* How well does thresholdout work on regression, or is it optimized for classification problems?
 
* There are a few hyperparameters used to determine the distribution of randomness in `thresholdOut`. I understand what they do mathematically, but it's not immediately clear how sensitive the method is to getting these just right. Ideally, there would be universally acceptable values there otherwise you'd just have to do another gridsearch to optimize `thresholdOut` before using `thresholdOut`
 
* The original paper(s) indicate that `thresholdOut` can be only be applied a finite number of times to a particular dataset (and the budget for number of allowed calls is determined by the hyperparameters mentioned above).
    * What happens if you exceed this number? Does overfitting sneak in, or something else? Is it a gradual decline or sudden, catastrophic breakdown?
    * What are best practices for knowing when a budget can 'reset'?
 
Overall, I'm intrigued by `thresholdOut` and I'm a little surprised I haven't heard much more about it since I started playing with it. I've spent a little time exploring the above questions in my linked notebooks, but if you have answers or hot thresholdOut news, feel free to send me an email.