---
title: SLM Chapter 2 Perceptron
date: 2019-11-06 08:56:00 -0500
author: Chonghan Chen
categories: [Study Notes, Statistical Learning Methods]
tags: [machine learning, notes, maths, english]
math: true
---




Essentially a hyperplane that divides the space into two classes.



## Properties

- A **linear**  and **binary** classifier.
- A **discriminative** (as apposed to generative) model.
- The hypothesis space is $\mathcal{F} = \{f\mid f(x) = w x + b; w, b \in \mathbb{R}^n\}$.
-  Can only be applied when training data is **Linearly separable**.



## Algorithm

### Prediction

For a data point, calculate $y = sign(w x + b)$ , where $sign$ is the sign function (outputting either -1 or 1, depending on the sign of its input).



### Loss Function

Loss is calculated based on the distance between a misclassified data point and the hyperplane:
$$
L(w, b) = -\sum_{(x_i,y_i) \in M}y_i(wx_i+b)
$$
where $M$ is the set of all misclassified data points.

Compared to the original distance measure
$$
Distance = \frac{1}{||w||}  |wx_i+b|
$$
the loss function

- Summed over the misclassified points of the entire data set.
- Multiplied a $y_i$ term to remove the absolute value operator, so that the loss function is differentiable.
- Removed $\frac{1}{||w||}$, and consider only the **functional margin** (see chapter 7).
### Optimization

The goal is to find parameters $w$ and $b$ such that
$$
w, b = \arg\min_{w,b}L(w, b)
$$
Each step, we update using gradient descent 
$$
w \leftarrow w - \eta \nabla_w L(w, b) \\
b \leftarrow b - \eta \nabla_b L(w, b)
$$
or 
$$
w \leftarrow w + \sum_{(x_i,y_i) \in M} \eta y_i x_i\\
b \leftarrow b + \sum_{(x_i,y_i) \in M} \eta y_i \\
$$
We can also consider updating one point at a time: for every step, we sample $(x_i, y_i)$ from M and do
$$
w \leftarrow w +  \eta y_i x_i\\
b \leftarrow b +  \eta y_i \\
$$
We repeat the update until convergence.



## Dual Perceptrons

If we repeat the above process, we will get our final (and optimal) parameters $\hat w$ and $\hat b$ calculated as follows
$$
\hat w = w_0 + \eta x_iy_i + \eta x_jy_j + \eta x_ky_k...  \\
\hat b = b_0 + \eta y_i + \eta y_j + \eta y_k...
$$
where $i, j, k, ...$ are each a misclassified data point that we choose in each step.

The above can be written as
$$
\hat w = w_0 + \sum_{i=1}^N \alpha_i x_iy_i  \\
\hat b = b_0 \sum_{i=1}^N \alpha_i y_i \\
$$
where $\alpha_i$ shows how often a point $(x_i, y_i)$ gets picked. Following our calculation, the value of each $\alpha$ should be some integer multiple of $\eta$.



### Idea
Suppose we have $D$ features and $N$ training samples.

By observation, we see that for each step, we need to find out misclassified points, which involves calculating $wx_i + b$ for $i = 1, 2,...,N$. The calculation takes $O(ND)$ time. 

If we use the dual form, each step we calculate $\alpha_i x_i y_i x_j + \alpha_i y_i$ for $i = 1, 2,...,N$. By pre-computing a Gram matrix $G = |x_i \cdot x_j |_{N \times N}$, the calculation will be driven down to $O(N)$ time.
### Prediction

Simply plug $w$ and $b$ in:
$$
f(x) = sign(\sum_{i=1}^N (\alpha_i x_i y_i x + \alpha_i y_i))
$$


### Optimization

We initialize $\alpha = (\alpha_1, \alpha_2, ..., \alpha_N)$ as an all-zero vector (meaning none of the data points have been picked).

For each misclassified point $(x_i, y_i)$, we do
$$
\alpha_i \leftarrow \alpha_i + \eta \\
b \leftarrow b + \eta y_i
$$


