---
title: SLM Chapter 1 Introduction 
date: 2019-11-03 03:46:00 -0500
author: Chonghan Chen
categories: [Study Notes, Statistical Learning Methods]
tags: [machine learning, notes, maths, english]
math: true
---


This is a series of learning notes that I made when reading the book *Statistical Learning Methods（统计学习方法)* by Li, Hang(李航).

## Elements of Statistical Learning

#### Model

A model $f$ is chosen from a hypothesis space $\mathcal{F} = \{f\mid Y = f_\theta(x), \theta \in \mathbb{R}^n\} $, or $\mathcal{F} = \{P\mid P = P_\theta(Y\mid X), \theta \in \mathbb{R}^n\} $ (i.e. a set of all possible models that is used), where $\theta$ is the parameter that determines a model $f$.

#### Strategy

The evaluation criterion of any candidate model $f$.

#### Algorithm

How to find the best model in $\mathcal{F}$ according to our strategy (i.e. how to solve the optimization problem).



## Types of Models

There are multiple ways to classify models.



### Probabilistic vs. Deterministic

These two can be converted to each other. Outputting the maximum of a probabilistic distribution gives a deterministic outcome, and normalizing a deterministic outcome gives a probabilistic prediciton.

##### Probabilistic

We learn a probability distribution as the final outcome. Specifically:

- In supervised learning, for each sample $x$, we learn a probability distribution of the prediction outcome $y$, i.e. we learn $P(y\mid x)$.
- In unsupervised learning (classification without knowing how many classes are there), we learn both $P(x\mid z)$ (probability of getting a sample given that it comes from a class, represented by $z$) and $P(z \mid x)$ (probability of the given sample belonging to a certain class $z$).

##### Deterministic

In both scenarios, we try to predict the most likely outcome (a certain output vector), instead of getting a probability distribution.

- In supervised learning, we learn a function $y = f(x)$
- In unsupervised learning, we learn $z = g(x)$



### Linear vs. Non-Linear

Easy.



### Parametric vs. Non-Parametric

##### Parametric Learning

The model is uniquely identified by all of its parameters (e.g. a neural network). It is fixed and finite-dimensional.

##### Non-Parametric Learning

The complexity, or the parameters of the model grows with the size of the dataset (e.g. SVM, KNN).





## Model Selection and Evaluation

**Expected risk** $E_p$ of a model is 

$$
\begin{align*}
R_{exp}(f) 
&= E_p[L(Y, f(X))] \\ 
&= \int{L(y, f(x))P(x, y)dxdy}
\end{align*}
$$

where $L$ is a loss function. There are many different choices for $L$, for example, L1 loss, L2 loss and log-likelihood loss. Our goal is to find a model $f$ such that

$$
f = \min_{f \in \mathcal{F}}{R_{exp}(f)}
$$

Yet, we cannot calculate $R_{exp}(f)$ directly since we do not know $P(x, y)$ (which actually requires learning). We cannot work out $P(x,y)$ first either, because $P(x,y)$  is the very thing that we need to learn.

To make the problem viable, we minimizes the **empirical risk** instead of the **Expected risk**:

$$
\begin{aligned}
f &= \min_{f \in \mathcal{F}}{R_{emp}(f)} \\ 
 &= \min_{f \in \mathcal{F}} \frac{1}{N} \sum^N_{i=1}{L(y_i, f(x_i))}
\end{aligned}
$$

When sample size $N$ becomes large enough, the empirical risk will approach expected risk.

However, when $N$ is small, we may consider minimizing the **structural risk** instead for better performance

$$
\begin{aligned}
f &= \min_{f \in \mathcal{F}}{R_{srm}(f)} \\ 
 &= \min_{f \in \mathcal{F}} \frac{1}{N} \sum^N_{i=1}{L(y_i, f(x_i))} + \lambda J(f)
\end{aligned}
$$

Where $J$ is a function that grows as the model $f$ becomes more complex.  E.g. when $f$ is parametrized by $\theta$, we can use $J(f) = \frac{1}{2}||\theta||^2$
### Generalization Error Bound

Let the real risk (**generalization error**) of our learned model $\hat f$ and its estimation (empirical risk) be

$$
R(\hat f) = E[L(Y, \hat f(X))]
$$

$$
\hat R(\hat f) = R_{emp}(\hat f) = \frac{1}{N} \sum^N_{i=1}{L(y_i, \hat f(x_i))}
$$

For a finite hypothesis space $\mathcal{F}$ of size $d$, we have 

$$
P(R(f) \le \hat R(f) + \epsilon(d, N, \delta)) >= 1 - \delta
$$

Where $N$ is the size of the training set, $0 \lt \delta \lt 1$, and $\epsilon = \sqrt{\frac{1}{2N}(\log{d} + \log{\frac{1}{\delta}})}$

(i.e. The generalization is bounded by the empirical risk. **The more training samples you have and the smaller the hypothesis space $\mathcal{F}$ is, the tighter the bound will be**.

The proof is done using the Hoeffding equation.





## Approaches: Generative vs. Discriminative

### Reference

https://www.cnblogs.com/rossiXYZ/p/12244760.html

### Generative Approach
Learns $P(Y|X)$ by using

$$
P(Y|X) = \frac{P(X,Y)}{P(X)}
$$

Once the model is learned, we can usually reconstruct $P(X, Y)$, i.e. we can generate samples that resembles what the model has seen. 



### Discriminative
Calculates directly $Y = f(x)$ or $P(Y|X)$.


