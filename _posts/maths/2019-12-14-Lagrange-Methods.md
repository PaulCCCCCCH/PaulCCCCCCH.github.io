---
title: Intuitions of Lagrange Methods Behind SVMs
date: 2019-12-14 21:12:00 -0500
author: Chonghan Chen
categories: [Study Notes, Maths]
tags: [notes, maths, optimization, tutorials]
math: true
---



# Lagrange Multipliers

## Set up
Consider the problem of minimizing a function $f(x)$ given $n$ constrants $h_1, h_2, ..., h_n$, i.e. find a $x = \{x_1, x_2, ..., x_k \}$ such that 

$$
\begin{aligned}
& x = \min_{\hat x} f(\hat x) \\
s.t. & \  h_i(x) = 0, i = 1, 2, ..., n
\end{aligned}
$$


## Intuition
We can try to imagine it in two dimensions, i.e. $k = 2$. We think of $f(x)$ as a basin and $h_i(x) = 1, 2, ...$ as contours. Imagine that you are trapped in an basin. There is toxic gas everywhere above you, and the higher you get, the sooner you will die. Therefore, in order to live longer, you have to descent as much as possible. This is the task of minimizing $f(x)$. On the other hand, you cannot just simply rush downwards. There is also monsters everywhere. You have to stay in the path lit by fire to protect yourself from being attacked. This is the task of satisfying the constraint $h_i(x) = 0$.

There is no light at all at the very bottom (level -500) of the basin, So, you start walking upwards unwillingly (you know are inhaling more and more toxic as you go upwards, but you have to do it so as to avoid monsters). When you are at level -400, you look around and try to find if there is any light at this level (i.e. does the path lit by light passes through level -400?). Unfortunatly, you don't see any. You keep going up. At level -300, you still don't see any, yet you know you are getting close. Not surprisingly, at level -233, you found that a tiny bit of the path just **barely touches** level -233. You finally reach the light, stay there and wait for rescue. Level -233 is the best you can hope for: there is no light at level -234, and the air is more toxic at level -232.

What is interesting is the word 'barely touch'. Actually, no matter how funny the path of fire may look like, when you get to the optimal level, you will always find that the path **barely touches** your level without passing through. If it passes through level -233, that means you can also get light at level -232, so you are not at the optimal level.

I hope this explains the follwing: at a minimum point of $f(x)$, the contour of $f(x)$ must be **tangent** to the contour of $h_i(x)$ (i.e. the path of fire barely touches the level).


Equivalently, we can say that the gradient of these two functions has to be parellel, i.e. for each $i$

$$
\nabla f(x) = \lambda_i \nabla h_i(x)
$$

The proof relies on something called `Implicit Function Theorem`. For now just hold the belief that the intuition is indeed correct. 


## Algorithm
To solve this kind of problems, we first construct a Lagrangian function.

$$
    L(x, \lambda) = f(x) + \sum_{i=1}^n \lambda_i h_i(x)
$$

The problem now is equivalent to

$$
    \min_{x, \lambda} L(x, \lambda)
$$


Then, we solve it with the following system of equations:

$$
\begin{cases}
    \nabla_x L(x, \lambda) = 0 \\
    h_i(x) = 0 \text{ for } i = 1, 2, ..., n
   
\end{cases}
$$

or more explicitly

$$
\begin{cases}
    \frac{\partial L(x, \lambda)}{\partial x_1} = 0 \\
    \frac{\partial L(x, \lambda)}{\partial x_2} = 0 \\
    ... \\
    \frac{\partial L(x, \lambda)}{\partial x_k} = 0 \\
    h_1(x) = 0 \\
    h_2(x) = 0 \\
    ... \\
    h_n(x) = 0 \\
   
\end{cases}
$$


# Generalized Lagrange Function

## References

https://zhuanlan.zhihu.com/p/38163970
https://www.cnblogs.com/ooon/p/5721119.html

## Set up
Consider the a more complex problem of minimizing a function $f(x)$ given inequality constraints $h_i \le 0$ instead of $h_i = 0$

$$
\begin{aligned}
& x = \min_{\hat x} f(\hat x) \\
s.t. & \  h_i(x) \le 0, i = 1, 2, ..., n \\
\end{aligned}
$$

## Intuition
Instead of a curve in 2d space (or a 'path of fire'), the constraint now becomes an area in 2d space (or an area covered by the 'holy aura').
The problem looks similar as before except that we need to consider two different cases:

### Case 1
The lowest point of the basin lies inside the area.
In this case, the constraints becomes useless, since you are going to go to the lowest point any way. You can forget about the constraint, and the problem becomes $\min_x f(x)$.

### Case 2
The lowest point of the basin lies outside the area. In this case, we need to find the lowest point in the basin that is covered by the 'holy aura'. Similar as the 'path of fire' case, we will still find that the optimal level 'barely touches' the boundary of the 'holy aura'. In other words, we still have $f(x)$ tangent to $h_i(x)$. Yet, this time we need to add some more constraints. We require that the gradients of $g_i(x)$ and $f(x)$ point to the opposite direction, i.e. 

$$
\begin{cases}
    -\nabla f(x) = \lambda \nabla_x g(x)  \\
    \lambda > 0
\end{cases}
$$

Think of it this way: the 'holy aura' decreases the 'power of darkness'. Outside the aura, the power of darkness is larger than 0, meaning monsters will thrive. Within the aura, the power of darkness is suppresses to negative, so the monsters cannot get in. At the edge of the aura, the power of darkness will be exactly 0. So, when you stand at the edge of the aura and look towards the bottom of the basin, you need to make sure that, as the level goes down, the power of darkness goes up. If the opposite is true (the power of darkness shrinks as you go down), you will be able to safely walk towards that direction and lower your position. This makes the point where you stood a sub-optimal point.


## Generalized Set Up
Consider the a more complex problem of minimizing a function $f(x)$ given not only equality constraints $h_i = 0$, but also inequality constrants $g_j <= 0$:

$$
\begin{aligned}
& x = \min_{\hat x} f(\hat x) \\
s.t. & \  h_i(x) = 0, i = 1, 2, ..., n \\
 & \  g_j(x) \le 0, j = 1, 2, ..., m \\
\end{aligned}
$$

## Algorithm
Define the lagrangian function 

$$
L(x, \lambda, \mu) = f(x) + \sum_{i=1}^{n} \lambda_i h_i(x) + \sum_{j=1}^{m} \mu_j g_j(x) 
$$

Then, we try to solve the following (a.k.a. KKT conditions. Given convex function $f(x)$ (and with some other constraints), $x$ is the minimum if and only if KKT conditions hold):

$$
\begin{cases}
    \nabla_x L(x, \lambda) = 0 \\
    h_i(x) = 0 \text{ for } i = 1, 2, ..., n \\
    g_j(x) \le 0 \text{ for } j = 1, 2, ..., m \\
    \mu_j \ge 0 \\
    \mu_jg_j(x) = 0,  j = 1,2, ..., m
   
\end{cases}
$$

Where condition 1 finds the minimum of $f(x)$, condition 2 and 3 are initial constraints, condition 4 ensures the gradients of $h_i(x)$ and $g_j(x)$ to point to opposite directions (see the 'holy aura' analogy, case 2).

Condition 5 follows the first case in our 'holy aura' analogy. When the minimum of $f(x)$ lies within the permissible area, i.e. $g_j(x) < 0$, we do not need to consider the constraints $g_j(x) \le 0$ anymore. We let $\mu_j = 0$, and $g_j(x)$ will automatically lose its power to constrain. When the minimum of $f(x)$ is on the edge of, or outside the permissible area, we have we are guaranteed to find the optima at $g(x) = 0$. In either case, we have $\mu_jg(j) = 0$.


## Duality

In fact, even if we have the KKT conditions, it can be hard to find a solution. To solve for $x$, We can combine the conditions together to form an equivalent problem with a single equation, and simply throw away other constraints:

$$
x = \min_{\hat x} \max_{\alpha, \beta} L(x, \alpha, \beta) \\
$$


Why can this problem be free of constraints? The idea is the following.
Think of it as a game of two players against each other. Player 1 tries to pick an $x$ that minimizes $L(x, \alpha, \beta)$, while player 2 tries to pick an $(\alpha,\beta)$ pair that maximizes it. 

If player 1 picks an $x$ that does not satisfy the constraints (e.g. by making $g_k(x) = 0.2$), player 2 will take advantage of it by making $\mu_k = +\infty$. The resulting function value will be $+\infty$, a crushing defeat for $x$. Therefore, player 1 has to pick $x$ wisely to satisfy the constraints.

Interestingly, if $L(x, \alpha, \beta)$ is convex and KKT conditions hold, we can simply switch the order of 'min' and 'max', and the result is guaranteed to be the same:

$$
\begin{aligned}
x &= \min_{\hat x} \max_{\alpha, \beta} L(x, \alpha, \beta) \\
&= \max_{\alpha, \beta} \min_{\hat x}  L(x, \alpha, \beta) \\
\end{aligned}
$$

This is called the dual problem of the original problem.
We can then solve the equation. We first try to find the solution to the inner minimization problem using

$$
\frac{\partial L(x, \alpha, \beta) }{\partial x} = 0
$$

Then, we use some iterative algorithm to solve the outer maximization problem. Suppose the solution we got in the previous step was $\psi(\alpha, \beta)$, we need to find

$$
\alpha^*, \beta^* = \arg \max_{\alpha, \beta} \psi(\alpha, \beta)  
$$

Finally, we substitute in the optimal $\alpha^* $ and $\beta^* $ and get our $x$ value:

$$
x = \psi(\alpha^*, \beta^*)
$$

## References
https://www.cnblogs.com/liaohuiqiang/p/7805954.html

https://www.bilibili.com/video/BV1aE411o7qd?p=32
