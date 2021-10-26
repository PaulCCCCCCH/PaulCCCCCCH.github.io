---
title: SLM Chapter 6 Logistic Regression
date: 2019-11-30 06:43:00 -0500
author: Chonghan Chen
categories: [Study Notes, Statistical Learning Methods]
tags: [machine learning, notes, maths, tutorials, english]
math: true
---


## Logistic Distribution

$$
F(x) = P(X \le x) = \frac{1}{1 + e^\frac{-(x - \mu)}{\gamma}} \\

f(x) = F'(x) = \frac{e^\frac{-(x - \mu)}{\gamma}}{\gamma(1 + e^\frac{-(x - \mu)}{\gamma}) ^ 2}
$$

where $\mu$ controls the center of the distribution, and $\gamma$ controls its sharpness (smaller $\gamma$ means shaper curve)

![Graph](/assets/img/statistical-learning-methods/LogisticDistribution.jpg)
_Logistic distribution and its CDF_

## Real Value to Probability
Suppose you want to calculate how likely a sample $x$ belongs to class $k$. You calculate $z = w * x + b$ for some weight $w$ and bias $b$. The problem is, $z$ will be in range $(-\infty, +\infty)$, while we require it to be in $(0, 1)$. Here is the solution

For some probability $p \in (0, 1)$, we calculate its **odds** as 

$$
odds(p) = \frac{p}{1-p}
$$

where $odd(p) \in (0, +\infty)$. In real life, when we describe, let's say, the chance of winning a lottery, we may say its '1 in 100000'. This is equivalent to saying that the probability $p$ of winning the lottery satisfies:

$$
\begin{aligned}
    odd(p) &= \frac{\text{chance of winning}}{\text{chance of losing}} \\
    &= \frac{1}{100000 - 1}
\end{aligned}
$$

We further map $(0, +\infty)$ to $(-\infty, +\infty)$ by taking its log value, resulting in the logit function

$$
    logit(p) = log \frac{p}{1-p}
$$


Finally, we can let 

$$
    logit(p) = w*x + b
$$


Solve for $p$, we get

$$
    p = \frac{1}{1 + e^{-(wx + b)}}
$$



# Chapter 6.2: Maximum Entropy Model
## References
https://repository.upenn.edu/cgi/viewcontent.cgi?article=1083&context=ircs_reports


## Idea
When a distribution can not be determined by the given evidence, we make a guess that maximizes the entropy of the distribution subject to the evidence as constraints.

Solving for the maximum entropy model is actually an constraint optimization problem: we would like to maximize the entropy of our model while satisfying the constraint that our model has to match our observation.

This is a discriminative model, i.e. estimates directly $p(y \mid x)$ (as opposed to generative model, see chapter 1 for comparison).

## Set Up for Target Function
We let the entropy of $P(Y \mid X)$ be:

$$
\begin{aligned}
    
H(P) &= E_xH(y \mid x) \\
&=  -\sum_{x, y} \tilde{p}(x)p(y \mid x)\ln(p(y \mid x)) \\
\end{aligned}
$$

where $\tilde{p}(x)$ is the probability of $x$ (calculated by counting).

$H(P)$ is then our maximization target.
## Set Up for Constraints
First, for an input sentence $x$ and label $y$, we define feature function $f(x, y)$ as

$$
f(x, y) = 
\begin{cases}
    1 \text{ if x and y satisfies some assertions} \\
    0 \text{ otherwise}
\end{cases}
$$

For a problem, we may have multiple feature functions $f_1, f_2, ..., f_n$.

For example, consider filling in the following blank
> I will \<BLANK> the piano when I get home.

In the training set, we see many different phrases. For example, 'The musician plays the piano', 'The shop sells the piano', etc. We may define the following feature functions:

$$
f_1(x, y) = 
\begin{cases}
    1 \text{ if word 'play' follows a person in x and y = 'play' } \\
    0 \text{ otherwise}
\end{cases}
$$

$$
f_2(x, y) = 
\begin{cases}
    1 \text{ if word 'play' follows a place name in x and y = 'sell' } \\
    0 \text{ otherwise}
\end{cases}
$$

It is then easy to see that the number of sentences where 'play' follows a person will then be $\sum_{x, y}f_1(x, y)$.

Therefore, the probability of observing such fact in the training set is

$$
    E_{\tilde{p}(x,y)}f_1(x, y) =  \sum_{x,y} \tilde{p}(x, y)f_1(x, y)
$$

We define our model (estimation) as $p(y \mid x)$. In our estimation, the probability of making such observation is

$$
    E_{p(x,y)}f_1(x, y) =  \sum_{x,y} \tilde{p}(x)p(y \mid x) f_1(x, y)
$$

Note that we factorize $p(x,y)$ as $\tilde{p}(x)p(y \mid x)$.

If our model is correct, we must have $E_{\tilde{p}(x,y)}f_1(x, y)$ =  $E_{p(x,y)}f_1(x, y)$ .

Therefore, in our model $p(y \mid x)$, for any feature function $f_i(x, y)$, we require that $E_{p(x,y)}$ is equal to $E_{\tilde{p}(x,y)}$, i.e. our model (estimation) matches our observation of the data. This will be our constraints for our model (together with the constraint that our estimated distribution should sum to 1).

## Optimization Problem
We need to optimize the entropy of our model $H(P)$ subject to constraints.
The optimization problem can be expressed as the following:

$$
\begin{aligned}
\max_P & 
    =  -\sum_{x, y} \tilde{p}(x)p(y \mid x)\ln(p(y \mid x)) \\
\text{s.t. } & E_p(f_i) = E_{\tilde{p}}(f_i), i = 1,2,...,n \\
& \sum_y P(y \mid x) = 1
\end{aligned}
$$

## Lagrange Function
We rewrite the above problem into a standard Lagrangian optimization problem

$$
\begin{aligned}
\min_P & 
    =  \sum_{x, y} \tilde{p}(x)p(y \mid x)\ln(p(y \mid x)) \\
\text{s.t. } & E_p(f_i) - E_{\tilde{p}}(f_i) = 0, i = 1,2,...,n \\
& \sum_y P(y \mid x) = 1
\end{aligned}
$$

We define the Lagrange function $L(p, w)$ as:

$$
L(P, w) = - H(P) + w_0(1 - \sum_y P(y \mid x)) + \sum_{i = 1}^n w_i(E_p(f_i) - E_{\tilde{p}}(f_i))
$$


and solve the following

$$
\frac{\partial L(P, w)}{\partial(P(y \mid x))} = 0
$$

See [link to the detailed explanation](/posts/Lagrange-Methods) on why we do this.


The solution tells us that our model $P_w(y \mid x)$ parametrized by $w$ will have the form

$$
P_w(y \mid x) = \frac{1}{z_w(x)} e^{\sum_i w_if_i(x, y)}
$$

where $Z_w(x)$ is a normalization term calculated as

$$
Z_w(x) = \sum_y e^{\sum_i w_i f_i(x, y)}
$$

The only problem left now is finding the optimal parameter $w^*$ that maximizes $P_w(y \mid x)$, i.e. 

$$
 w^* = \arg \max_w P_w(y \mid x)
$$

which we leave to the all-mighty gradient-descent method.

