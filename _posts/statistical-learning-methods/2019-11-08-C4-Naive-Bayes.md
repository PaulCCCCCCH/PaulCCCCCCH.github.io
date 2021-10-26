---
title: SLM Chapter 4 Naive Bayes
date: 2019-11-08 06:18:00 -0500
author: Chonghan Chen
categories: [Study Notes, Statistical Learning Methods]
tags: [machine learning, notes, maths, english]
math: true
---



A generative learning model.



## Assumption

Given a class label, each feature of a data point should be independently distributed.

Specifically, for any data point $x_i=(x_i^{(1)}, x_i^{(2)},..., x_i^{(n)})$ and any class $c_k \in \{c_1, c_2, ..., c_K\} $

$$
P(x_i|c_k) = P(x_i^{(1)}, x_i^{(2)},..., x_i^{(n)} \mid c_k)
$$

should factorize to

$$
\begin{align}
P(x_i|c_k) &= P(x_i^{(1)} \mid c_k)P(x_i^{(2)} \mid c_k)...P(x_i^{(n)} \mid c_k) \\
&= \prod_{j=1}^{n}P(x_i^{(j)} \mid c_k)  
\end{align}
$$


## Algorithm

Following the assumption and basic Bayesian rule, we derive the following

$$
\begin{align}
P(c_k|x) &= \frac{P(x \mid c_k) P(c_k)}{P(x)} \\
&= \frac{\prod_{j=1}^{n} P(x^{(j)} \mid c_k) P(c_k)}{\sum_{k=1}^K P(x, c_k) P(c_k)} \\
&= \frac{\prod_{j=1}^{n} P(x^{(j)} \mid c_k) P(c_k)}{\sum_{k=1}^K \prod_{j=1}^{n}P(x^{(j)} \mid c_k) P(c_k)}
\end{align}
$$


For $k=1,2,...,K$, we estimate $P(c_k)$ simply by counting the training set

$$
P(c_k) = \frac{\text{samples in class k}}{\text{training set size}}
$$

We could also model $P(c_k)$ as a Dirichlet distribution (a multi-dimensional beta distribution).

We then model $P(x \mid c_k)$ as well. One way of doing it when the feature space is discrete is

$$
P(x \mid c_k) = \frac{\text{number of samples in class k that has a particular value}}{\text{number of samples in class k}}
$$

Plug everything in, and can then make our prediction by calculating

$$
y = \arg \max_{c_k} P(c_k \mid x) \\
= \arg \max_{c_k} \prod_{j=1}^{n} P(x^{(j)} \mid c_k) P(c_k)
$$


### Laplacian smoothing

$P(c_k)$ and $P(x \mid c_k)$ are likely to be zero if some particular value is never observed. This happens when there are too many classes and features while having too little training samples.

To solve this issue, we add a smoothing term $\lambda > 0$ to the numerator. We add some multiple of $\lambda$ to the denominator to normalize the probability.

