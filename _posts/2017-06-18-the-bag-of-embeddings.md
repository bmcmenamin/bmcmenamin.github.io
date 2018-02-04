---
layout: post
title: "The Bag of Embeddings"
author: "Brenton McMenamin"
tags: [math]
---

Word2vec is great. If you're the kind of person to be reading a post here, you've probably already seen demos showing how the word embeddings created by word2vec preserve semantic relationships between words. But if you haven't, here's what you need to know about word2vec for now: Word2vec is an unsupervised method that uses a large text corpus to calculate "word embeddings". These embeddings take a word and map it to a high-dimensional numerical vector, such that words with similar semantic meanings are mapped to vectors that are close together.

After reading about word2vec, I really wanted to play around with it so I downloaded the large [pre-trained 300-dimensional embeddings](https://github.com/mmihaltz/word2vec-GoogleNews-vectors) made from a huge Google News corpus. But, I immediately ran into a problem -- the embeddings made it easy to compare similarity between individual words and phrases, but how do you scale this up to compare the similarity between entire documents?

A common solution is to create a document-level embedding by simply averaging all the word-embeddings in a document. This results in a single embedding vector for each document that can be used to measure document similarity, and it works surprisingly well in many situations. However, I was worried that if a document is long enough to address multiple distinct topics, the 'average semantic content' could actually be misleading (in the same way that taking the mean of a bimodal distribution can be misleading). So, I decided to look into alternative methods for document-comparison with word embeddings.

## Comparing documents using word2vec embeddings

One solution is to explicitly train higher-order embeddings for entire documents (_e.g._, doc2vec), but this requires big sets of training data that may not be available for many applications. Instead, I'd like to find a way to re-use word-level embeddings to compare documents.

### Word Mover Distance

The [Word Mover's Distance (WMD)](http://mkusner.github.io/publications/WMD.pdf) is a method for comparing the individual word embeddings in a pair of documents that is nicely described [this blog post](http://vene.ro/blog/word-movers-distance-in-python.html). The WMD is calculated by taking the sets of word embeddings from two documents, and then measuring how far it'd have to 'move' the embeddings from one document to align it with the other. Unfortunately, the runtime is $O(p^3 \log p)$, where $p$ is the number of unique words, which means that the WMD is very slow for large documents.

### Bag of Embeddings

I liked the concept of document similarity behind the WMD, where two documents were considered similar if the set of embeddings from each had similar distributions across the semantic space. But the runtime copmlexity is pretty bad, so I started thinking about faster ways to do a similar calculation and developed an approach that I'm calling the Bag of Embeddings (BoE). The name comes from the fact that, as you'll see below, this is a generalization of the Bag of Words (BoW) model that has been used in traditional natural language processing ([relevant link](https://www.youtube.com/watch?v=QE5D2hJhacU)).


#### Thinking about embeddings as distributions rather than points

We're normally told that an $n$-dimensional embedding maps a word $i$ to a single point in semantic space, $\boldsymbol{w}_i$. The semantic similarity of two words can then be calculated the distance between them in semantic space.

Instead, let's assume that the embedding maps a word to a radial basis function. So, instead of word $i$ being represented as just a single point, the word is represented as a probability distribution that covers the *entire* semantic space. Specifically, this it is mapped to the multivariate normal distribution with spherical covariance, $\mathcal{N}(\boldsymbol{w}_i, {\boldsymbol{I}s^2})$. Now, when we're given a second embedded word, $\mathcal{N}(\boldsymbol{w}_j, {\boldsymbol{I}s^2})$, we can measure their similarity using the "overlap" between these two distributions to get a sense for how close the two points are. There are several ways of measuring the overlap of two distributions, but we're going with a simple one -- integrate over the product of these two distributions -- because the mathemagical properties of Gaussians (and our assumption of spherical covariance) make it simple:

$$
\begin{align}

\mathrm{overlap}_{s}(\boldsymbol{w}_i, \boldsymbol{w}_j) &=
{(2{\pi}{s^2})^{-\frac{1}{2}n}}
\exp\left(
    {-\frac{1}{2s^2}}
    {(\boldsymbol{w}_i - \boldsymbol{w}_j)^\top}
    {(\boldsymbol{w}_i - \boldsymbol{w}_j)}
\right) \\

&=
{(2{\pi}{s^2})^{-\frac{1}{2}n}}
\exp\left(
    {\frac{1}{2}}
\right)
\exp\left(
    \frac{-\lVert \boldsymbol{w}_i - \boldsymbol{w}_j \rVert}{s^2}
\right) \\

&=
c
\exp\left(
    \frac{-\lVert \boldsymbol{w}_i - \boldsymbol{w}_j \rVert}{s^2}
\right)

\end{align}
$$


Which we can simplify to just:

$$
\mathrm{overlap}_{s}(\boldsymbol{w}_i, \boldsymbol{w}_j) \propto

\exp\left(
    \frac{-\lVert \boldsymbol{w}_i - \boldsymbol{w}_j \rVert}{s^2}
\right)
$$


This is just a small non-linearity wrapped around the pointwise distance we'd use in a more typical application of word embeddings. In fact, if $s^2$ is sufficiently large, the range of values inside the exponential will be restricted to a small range that can be approximated be a linear function and the overlap will be proportional to the typical distance-based score typically used, $-\lVert \boldsymbol{w}_i - \boldsymbol{w}_j \rVert$.


So why does it matter if we apply this non-linearity to our distance measure or not?

Well, now that we understand how to work with words that are represented as probability distributions across semantic space, we easily build document representations that are probability distributions across semantic space. Given a document, $\boldsymbol{D}$, that contains $\lvert \boldsymbol{D} \rvert$ unique words, we can represent it as the probability density function, $\mathcal{D}$, where:

$$
\mathcal{D} =
    \frac{1}{\mathrm{crossmag}(\boldsymbol{D}, \boldsymbol{D})}
    \sum_{\boldsymbol{w} \in \boldsymbol{D}}
    \mathcal{N}(\boldsymbol{w}, {\boldsymbol{I}s^2})
$$

The denominator of the normalization term, $\mathrm{crossmag}(\boldsymbol{D}, \boldsymbol{D})$, is the cross-magnitude of $\boldsymbol{D}$ with itself where "cross-magnitude" is the average overalp between every pair of words in two documents:

$$
\mathrm{crossmag}(\boldsymbol{D}_1, \boldsymbol{D}_2) =
    \frac{1}{\lvert \boldsymbol{D}_1 \rvert \times \lvert \boldsymbol{D}_2 \rvert}
    \sum_{i \in {\boldsymbol{D}_1},
          j \in {\boldsymbol{D}_2}
    }
    \mathrm{overlap}(i, j)
$$


Now we can now extend the concept of overalp from a pair of words to pairs of documents.


I'm not going to work through all the math here, but it turns out that you can simply decompose each distribution into the Gaussians it's made up of and do lots of pairwise overlap calculations.

For a pair documents, \mathcal{D}_1 and \mathcal{D}_2, the document-level overlap score is calculated as:

$$
\mathrm{BoE Overlap}(\boldsymbol{D}_1, \boldsymbol{D}_2) =
    \frac{
        \mathrm{crossmag}(\boldsymbol{D}_1, \boldsymbol{D}_2)
    }{
        \sqrt{
            \mathrm{crossmag}(\boldsymbol{D}_1, \boldsymbol{D}_1) \times
            \mathrm{crossmag}(\boldsymbol{D}_2, \boldsymbol{D}_2)
        }   
    }
$$



And the best part is that the run time is significantly better than WMD because it runs in quadratic time! The runtime for $\mathrm{crossmag}(\boldsymbol{D}_1, \boldsymbol{D}_2)$ is $O(\lvert \boldsymbol{D}_1 \rvert \times \lvert \boldsymbol{D}_2 \rvert)$, so the runtime for $\mathrm{BoE Overlap}(\boldsymbol{D}_1, \boldsymbol{D}_2)$ is 
$O(\lvert \boldsymbol{D}_1 \rvert \times \lvert \boldsymbol{D}_2 \rvert) +
O(\lvert \boldsymbol{D}_1 \rvert ^2) +
O(\lvert \boldsymbol{D}_1 \rvert ^2) \to
O( \max[{\lvert \boldsymbol{D}_1 \rvert^2, \lVert \boldsymbol{D}_2 \rVert^2}])
$. Whereas WMD ran in in  $O(p^3 \log p)$ where $ p \ge \max[{\lvert \boldsymbol{D}_1 \rvert, \lvert \boldsymbol{D}_2 \rvert}]$.


### How does Bag of Embeddings relate to the good ol' fashioned Bag of Words?

It turns out that normal bag of words (BoW) is actually a special case of the BoE method.

For some background, here's a description of how normal BoW works:

Create a vocabulary of $n_{vocab}$ words from your corpus. Now each document is represented by a length-$n_{vocab}$ vector, $\boldsymbol{d}$, that contains counts of how many times each word from the vocabulary occurred in the document. The similarity between documents $a, b$ can be measured by the distance between $\boldsymbol{d}_a, \boldsymbol{d}_b$ in a manner that reflect the proportion of words shared by the two documents.

You can get equivalent results using BoE if your $n_{vocab}$ words were embedded orthogonally in $n_{vocab}$-dimensional space (_e.g.,_ the embedding vector for the $i^{th}$ word is a vector of all 0's with a 1 at the $i^{th}$ element). This would result in a "semantic space" that behaves equivalently to simple string matching -- words are either exactly the same or completely distinct. This will result in:

* $\mathrm{crossmag}(\boldsymbol{D}_1, \boldsymbol{D}_2)$ returns the proportion of words shared by the two documents

* $\mathrm{crossmag}(\boldsymbol{D}, \boldsymbol{D})$ returns $\frac{1}{\lvert\boldsymbol{D}\rvert}$

Combine these two facts, and you'll see that $\mathrm{BoE Overlap}(\boldsymbol{D}_1, \boldsymbol{D}_2)$ will put the overlap measure in the numerator and some normalization terms in the denominator that would result in penalize short documents that only have a small number of words with lower 'overlap' scores.


## How does BoE compare to the average embedding on real data?


In the notebook [Find similar questions.ipynb](https://github.com/bmcmenamin/word2vec_advice/blob/master/Find%20similar%20questions.ipynb), I walk through some examples comparing how the BoE similarity compares to the similarity derived from average word embeddings using my [Dear Abby](https://bmcmenamin.github.io/2017/06/17/dear-abby-dataset.html) dataset.


## Relationship between Average Embedding Similarity and BoE Similarity

I took a reader-submitted question at random, and measured how similar it was to every other question using four different similarity metrics:
* Average Embedding Similarity (`avg_sim1`)
* BoE with $s=1.0$ (`boe_s1_sim`)
* BoE with $s=0.1$ (`boe_s01_sim`)
* BoE with $s=0.01$ (`boe_s001_sim`)

This plots below show the relationship between Average Embedding (`avg_sim`) and BoE similarities (`boe_*_sim`) over a range of $s$ values. The plots on the left show the relationship between actual distances, the plots on the right have been rank-transformed to more clearly show the relationship by removing skew and outliers from the distributions. As predicted, these two metrics are virtually identical for large values of $s$, and the relationship gets weaker as $s$ gets smaller.

|| Average Embedding Similarity vs. BoE Similarity | Rank-transformed similarities|
|:---:|:---:|:---:|
|$s=1.0$|<img src="/figs/boe/avgsim_boes1.png" width="300">|<img src="/figs/boe/avgsim_boes1_rank.png" width="300">|
|$s=0.1$|<img src="/figs/boe/avgsim_boes01.png" width="300">|<img src="/figs/boe/avgsim_boes01_rank.png" width="300">|
|$s=0.01$|<img src="/figs/boe/avgsim_boes001.png" width="300">|<img src="/figs/boe/avgsim_boes001_rank.png" width="300">|


## Qualitative performance

So, these measures start to differ when $s$ gets small, but which metric is better? To figure that out, I'm going to look at the highest-recommended match according to each metric and do some qualitative assessment.

For reference, this is the question that was randomly selected to serve as the search target:

> DEAR ABBY: My problem concerns Christmas gift-giving to my children and/or grandchildren. Their circumstances are not alike, and I want to be fair. One daughter is divorced with one child. One daughter is married with no children. One daughter and her husband have two children. My question is -- should I allot a certain amount of money for each individual, or each family unit? And should the fact that one daughter has less than the others enter into the picture? Is there a fair solution? -- CONCERNED IN FLORIDA

### Most similar questions according to Average Embedding

> DEAR ABBY: I am 49, divorced, and partially living with a 67-year-old man. He has been divorced nearly 10 years. He and his wife divorced because he had fathered a child by another woman. He never married this woman, but he does take care of the financial obligations for mother and child. He and I fight a lot because of his involvement with his ex-wife and ex-mistress. We have never spent a major holiday together because his adult children have all the family dinners, and I am not "family." I share my home, cabin, family, friends and vacations with him. Yet he thinks he "owes" the holidays and birthdays to his family. Last year we planned to have his birthday party at his house. We invited his children, but not the ex-wife, ex-mistress or child. Well, no one showed up. They all blamed me for the exclusion. I'm fairly intelligent and own my own business. Deep down, I want to get rid of him, but like the old rhyme says, "When it's good, it's very, very good -- and when it's bad, it's horrid." By the way, he recently went on a cruise with his ex-wife, and when they go to family gatherings, they share a room. I am supposed to understand that it's "family." Well, I am sick of this sick family. He says he would take me to family events, but the children don't want me. He says he doesn't want to hurt them any more than he already has, because the affair that produced the child lasted for 12 years of his marriage. He argues that as long as we spend weekdays together he should be able to spend the dozen-or-so birthdays and holidays with them. I think I should be included in family events or at least considered. Am I wrong? -- HAD ENOUGH IN MINNESOTA


> DEAR ABBY: This is in response to "Sad Grandpa" who, 38 years ago, married a widow with seven children. He now wants to remarry after her death, and his stepchildren are upset about it. Perhaps his stepchildren are afraid not only of losing their stepfather, but also any property or belongings (monetary or not) that their biological father and mother would have left them. I don't mean to make them sound greedy, but we all know from the letters you print that people who remarry later in life when their children are grown often forget that what they and their first spouse accumulated should be shared with their children. When parents remarry and are outlived by their new spouse, many times everything goes to the new spouse's family. And the children feel resentful and abandoned when nothing of their original family, monetary or not, is left for them. Many clergy today require young couples about to marry to attend premarital counseling, but this is often waived for mature couples who have been married before. Perhaps these couples should also attend counseling which, among other things, would cover disposition of property and how each other's heirs will be remembered in their wills. -- ELYN KIRCHNER, MINNEAPOLIS

### Most similar questions according to BoE $s=1.0$

> DEAR ABBY: I am 49, divorced, and partially living with a 67-year-old man. He has been divorced nearly 10 years. He and his wife divorced because he had fathered a child by another woman. He never married this woman, but he does take care of the financial obligations for mother and child. He and I fight a lot because of his involvement with his ex-wife and ex-mistress. We have never spent a major holiday together because his adult children have all the family dinners, and I am not "family." I share my home, cabin, family, friends and vacations with him. Yet he thinks he "owes" the holidays and birthdays to his family. Last year we planned to have his birthday party at his house. We invited his children, but not the ex-wife, ex-mistress or child. Well, no one showed up. They all blamed me for the exclusion. I'm fairly intelligent and own my own business. Deep down, I want to get rid of him, but like the old rhyme says, "When it's good, it's very, very good -- and when it's bad, it's horrid." By the way, he recently went on a cruise with his ex-wife, and when they go to family gatherings, they share a room. I am supposed to understand that it's "family." Well, I am sick of this sick family. He says he would take me to family events, but the children don't want me. He says he doesn't want to hurt them any more than he already has, because the affair that produced the child lasted for 12 years of his marriage. He argues that as long as we spend weekdays together he should be able to spend the dozen-or-so birthdays and holidays with them. I think I should be included in family events or at least considered. Am I wrong? -- HAD ENOUGH IN MINNESOTA


> DEAR ABBY: A year ago, I ended a turbulent five-year relationship with my boyfriend, "Alex," that resulted in a special-needs child. Alex is not living in reality when it comes to our daughter's disabilities, and his family is not present in her life. Our daughter, "Meghan," spent months in the hospital before she was healthy enough to come home, and Alex's family visited only a few times. I have tried to resolve the issues with Alex's family so our daughter can have a relationship with them, but it is still one-sided. Meghan's paternal family will send a present for her birthday or Christmas, but they spend no time with her. They have other grandchildren in other states that his mother drives hours to see, but she won't drive five minutes to see my daughter. I'd like to start rejecting the gifts they send Meghan with a note explaining why. I find it disturbing that they'll spend money on my child, but are unwilling to spend time with her. I feel the gifts are a payoff. I don't want Meghan to feel like the odd man out when she's old enough to realize how she is treated compared to the other grandchildren. Abby, what are your thoughts? -- END OF MY ROPE

### Most similar questions according to BoE $s=0.1$


> DEAR ABBY: My husband, "Todd," and I have been happily married for four years and together for six. We have a daughter (mine from a former marriage) and a beautiful little boy together. I know beyond a shadow of a doubt that Todd loves both children equally. Despite some tough financial times over the past two years, we are a happy family. Our problem? Todd's mother. She's a negative, bitter woman who insists she "can't possibly" show our daughter the same love she shows our son. She sends affectionate notes to our son, none to our daughter. She shops at discount stores for our daughter and only the best shops for our son. She sent our son a beautiful handmade toy and our daughter a pencil -- yes, a pencil! Please understand this isn't about gifts or the amount she spends. It's about the obvious disparity. Even worse, she's always saying that Todd couldn't possibly love our daughter the way he does our son. Need I tell you the damage this has already done to our daughter? We are at our wit's end. Todd is ready to just walk away from his mother. I know we can't change the way she feels, but are we wrong to insist that she not show it so openly to our daughter? Help. Please. -- READY TO WALK AWAY

> DEAR ABBY: Both my grown daughters work, and I take care of their daughters for them. Granted, I have them only a few hours a day, but I still have to feed them and give them snacks and juice. All I ask in return is $20 for each child every two weeks to help pay for the food and beverages. My older daughter says it's definitely worth it to her, as it would cost far more for someone else to care for her daughter. My younger daughter and her husband, however, are throwing a fit. They insist that a grandmother should never charge money to watch her own grandchild. I also watch them on weekends and barely get a thank-you. What is your opinion, Abby? Am I ... A GRANDMA OR A DOORMAT?



### Most similar questions according to BoE $s=0.01$

> DEAR ABBY: My husband, "Todd," and I have been happily married for four years and together for six. We have a daughter (mine from a former marriage) and a beautiful little boy together. I know beyond a shadow of a doubt that Todd loves both children equally. Despite some tough financial times over the past two years, we are a happy family. Our problem? Todd's mother. She's a negative, bitter woman who insists she "can't possibly" show our daughter the same love she shows our son. She sends affectionate notes to our son, none to our daughter. She shops at discount stores for our daughter and only the best shops for our son. She sent our son a beautiful handmade toy and our daughter a pencil -- yes, a pencil! Please understand this isn't about gifts or the amount she spends. It's about the obvious disparity. Even worse, she's always saying that Todd couldn't possibly love our daughter the way he does our son. Need I tell you the damage this has already done to our daughter? We are at our wit's end. Todd is ready to just walk away from his mother. I know we can't change the way she feels, but are we wrong to insist that she not show it so openly to our daughter? Help. Please. -- READY TO WALK AWAY

> DEAR ABBY: What do you do when your best friend knowingly names her dog the name that you had picked for your future daughter (should there be a daughter)? Am I being silly to feel upset? -- HURT IN MINNEAPOLIS



## Conclusions

It may just be confirmation bias speaking, but to me it looks like BoE with intermediate levels of $s$ does the best job finding relevant questions. I haven't thought about whether there's a principled way to select the best $s$ value, but there probably is.

While you're at it, here's a laundry list of other things that could be done to improve the bag of embeddings:
* non-spherical covariance
* putting a tf-idf type correction into semantic space
* varying covariance parameters for each word