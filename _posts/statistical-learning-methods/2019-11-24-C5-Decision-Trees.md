---
title: SLM Chapter 5 Decision Trees
date: 2019-11-24 21:20:00 -0500
author: Chonghan Chen
categories: [Study Notes, Statistical Learning Methods]
tags: [machine learning, notes, maths, tutorials, english]
math: true
---






## Idea

Each node is a decision maker. It splits a set of data into several sub-sets according to some criterion, and passes each sub-group to its corresponding child node. A class label is given to each leaf node, where no further decisions are made. When predicting the label of a new sample, we let the sample travel from the root node to a leaf node, following the decision rules along its way. When it reaches a leaf node, we assign the label of the leaf node to this sample.

The idea is demonstrated by the following pseudo code:

```python
def buildNode(dataset: List[Sample]):

    # We first comeup with a criterion with the dataset
    criterion = get_criterion(dataset) # Criterion: Sample -> Int
    
        
    # If we find out that we don't need to further split the dataset,
    # we may stop here and return a leaf node.
    stop, label = should_stop(dataset, criterion)
    if stop:
        return Node(None, label=label)
    
    
    # Then we create one subset for each criterion
    subsets = [[] for _ in range(criterion.k_children)]
    for sample in dataset:
        kth = criterion(sample)
        subsets[kth].append(sample)
    
    # Recursively build child nodes
	children = [build_node(subsets[i]) for i in range(k_children)]
    
    # Call the constructor to create current node
    node = Node(children, criterion)
    
    return node
```

## Intuitions

A decision tree can be seen as a series of if-then statements.

It is highly interpretable.

It trains very quickly.

## Background

### Conditional Entropy
Conditional entropy $H(Y \mid X)$ is defined as **the expectation of the entropy of $Y$ when X is given a specific value.** Mathematically:


$$
\begin{align}
H(Y \mid X) &= \sum_{x \in X} p(x)H(Y \mid X = x) \\
&= - \sum_{x \in X} p(x) \sum_{y \in Y} p(y \mid x) \log p(y \mid x) \\
&= - \sum_{x \in X} \sum_{y \in Y} p(x,y) \log p(y \mid x) \\
&= - \sum_{x \in X} \sum_{y \in Y} p(x,y) \log \frac{p(x, y)}{p(x)} \\
&= - \sum_{x \in X} \sum_{y \in Y} p(x,y) \log p(x, y) - (- \sum_{x \in X} \sum_{y \in Y} p(x,y) \log p(x)) \\
&= - \sum_{x \in X} \sum_{y \in Y} p(x,y) \log p(x, y) - (- \sum_{x \in X} p(x) \log p(x)) \\
&= H(X, Y) - H(X)
\end{align}
$$


## Components

Decision tree learning usually involves three steps: 

- feature selection
- decision tree generation
- pruning




## Feature Selection

At each step, we need to decide how to divide a data set into subsets, i.e. implementing `get_criterion` function used in the pseudo code above. Usually, a criterion is something like:

```python
def someExampleCriterion(sample):
    if sample.x < 1:
        return 1
    elif 1 <= sample.x < 5:
        return 2
    else: # sample >= 5:
        return 3
```

Therefore, we need to decide the following:

- **How many subsets do we need to create?** There are 3 different return values for this example criterion, but why not make it 4, 5 or even 42?
- **Which feature do we use?** In the example, we use `sample.x`, but why not `sample.y`?
- **What are the points of split?** We use 1 and 5 in the example, but why not 2 and 4, or 10 and 100?

A criterion will be uniquely determined by the three questions above. The thing is, how do we know if an answer to the questions is good or bad?



### Maximizing Information Gain

##### Definition

We can make a judgment by using **information gain**. We consider an answer with a larger information gain to be better.

For a given data set $D$ and an answer (i.e a criterion) $A$, information gain is defined as follows:

$$
\begin{aligned}
g(D, A) &= H(D) - H(D \mid A) \\
&= H(D) - \sum_{i=1}^N \frac{|D_i|}{|D|} H(D_i)
\end{aligned}
$$

where 

- $N$ is the number of subsets produced by the criterion $A$, 

- $D_i$ is the $i$th of the $N$ subsets

- $H(D)$ is the empirical entropy of the data set $D$ calculated as 

  $$
  \begin{aligned}
  H(D) &= - \sum_{k=1}^{K} \hat p_k  \log \hat p_k \\
  &= - \sum_{k=1}^{K} \frac{|C_k|}{|D|} \log \frac{|C_k|}{|D|}
  \end{aligned}
  $$

  Note that we use $\hat p_k$ to represent the empirical probability that a sample in $D$ has a label $k$. This is calculated by counting, i.e. dividing  $|C_k|$ (the number of samples labeled $k$) by $|D|$ (the total number of samples in $D$).
Intuitively, the best split of a data set $D$ should reduce the chaos (entropy) of the data set by the largest amount.



##### Finding a split $A$

Following the discussion above, our goal is to find a split $A$ such that

$$
A = \arg \max_{\hat A \in S} g(D, \hat A)
$$

where $S$ is the search space for $A$.

We can first design such a search space. For example, we may do this with a grid search:

```python
def get_criterion():
    best_split = None
    for feature in all_features:
        for num_child in range...:
            for split_point in range...
                calculate_information_gain(...)
                best_split = update_best(...)
                
    def criterion(sample):
        ...split using the best_split variable
       
    return criterion
```

##### Problem
Interestingly, the resulting criterion $A$ will always favor the ones with a large number of children. Consider a criterion that creates a subset for every single sample. This is guaranteed to produce a maximum information gain by completely eliminating the entropy after the split, i.e. $H(D \mid A) = 1\log1 \sum0\log0 = 0$.

This is meaning less, and we need to punish the model for creating too many subsets.



### Maximizing Information Gain Ratio

Instead of finding A with the following

$$
A = \arg \max_{\hat A \in S} g(D, \hat A)
$$

we use

$$
\begin{aligned}
A &= \arg \max_{\hat A \in S} g_R(D, \hat A) \\
&= \frac{g(D, A)}{H_A(D)} \\
&= \frac{g(D, A)}{- \sum_{i=1}^N \frac{|D_i|}{|D|} \log \frac{|D_i|}{|D|}}
\end{aligned}
$$

where $n$ is the number of subsets of $D$ created by criterion $A$, and $D_i$ is the $i$th subset.
### Minimizing Gini Index

This is used by CART algorithm during classification to find an optimal criterion.

Just like entropy, Gini index measures how chaotic a probability distribution is. 

For a classification problem with $K$ classes, if we let $p_k$ be the probability for a sample to be in class $k$, and $p = \{p_1, p_2,...,p_k\}$, the Gini index will be

$$
\text{Gini}(p) = \sum_{k=1}^{K}p_k(1-p_k)
$$

In fact, Gini index is an approximation for the entropy if we consider substituting $log$ with its first order Taylor expansion (i.e. $\ln (x) \approx x - 1$ at $x \approx 1$)

$$
\begin{aligned}
H(p) &= - \sum_{k=1}^{K} p_k  \log p_k \\ 
& \approx -\sum_{k=1}^{K} p_k  f(p_k) \\ 
& \approx  - \sum_{k=1}^{K} p_k (f(1) + f'(1)(p_k - 1)) \\
&=  - \sum_{k=1}^{K} p_k(p_k -1) \\
&= \sum_{k=1}^{K} p_k(1 - p_k) \\
&= \text{Gini(p)}
\end{aligned}
$$

We select a feature $A$ as 

$$
\begin{aligned}
A &= \arg \min_{\hat A \in S} \text{Gini}(D, \hat A) \\
&= \arg \min_{\hat A \in S} \sum_{i=1}^N \frac{|D_i|}{|D|} \text{Gini}(D_i) \\

\end{aligned}
$$

where $N$ is the number of subsets of $D$ created by criterion $A$, and $D_i$ is the $i$th subset.

Gini index can be calculated faster than entropy, since it does not involve $\log$ operators.
### Minimizing Square Loss

This is used by CART algorithm during regression.

Recall the fact that a decision tree divides the sample space $R$ into multiple sub-regions $R_1, R_2, ..., R_M$, with each node representing one region. During regression, instead of assigning a class label to each region $R_m$, we assign a continuous value $c_m$ to the region, so that when making predictions, we predict output $c_m$ for every sample in region $R_m$.

 We can then find the best split by minimizing the square loss:

$$
\begin{aligned}
A &= \arg \min_{\hat A \in S} \text{SquareLoss}(D, \hat A) \\
&= \arg \min_{\hat A \in S} \sum_{i=1}^N (y_i - f(x_i))^2  \\

&= \arg \min_{\hat A \in S} \sum_{i=1}^N (y_i - \sum_{m=1}^M c_mI(x \in R_m))^2  \\

\end{aligned}
$$

where $I$ is the indicator function, $N$ is the number of samples, $M$ is the total number of subregions, and $c_m$ can be calculated as the average output $\bar y$ of all samples in region $m$

If we limit the decision tree to be a binary tree, i.e. $M = 2$, then we only need to iterate through the split point and the features to use.
## Decision Tree Generation

See the pseudo code in the *Idea* section.

We further fill in the details of the `shouldStop` function:

```python
def should_stop(dataset, criterion):
    if all samples in dataset have same label:
        return True, label
    if calc_information_gain(dataset, criterion) < 0.01 # Just an example. The threshold can be passed in as a parameter 
  		return True, max_vote(dataset)
    
    return False, None
```





## Decision Tree Pruning
We define a loss function $C_{\alpha}(T)$ for a decision tree $T$

$$
\begin{aligned}
C_{\alpha}(T) &= C(T) + \alpha|T| \\
&= \sum_{t=1}^{|T|}N_tH_t(T) + \alpha |T|
\end{aligned}
$$

where $|T|$ is the number of leaf nodes, $N_t$ is the number of nodes in leaf node $t$ and $H_t$ is the empirical entropy at node $t$. 
For each leaf node, we calculate the loss of the entire tree before and after the node is merged into its parent. If we result in a smaller loss, we do the pruning. We repeat the process until there is no leaf node that can be pruned.
## Algorithms

### ID3

See the pseudo code. For each node, it chooses a split that maximizes the **information gain**.

### CD4.5

Same as ID3 except that it maximizes **information gain ratio** instead.

### CART

Short for **classification and regression tree**. 

It is a binary tree.

It uses **Gini index** during classification, and **Square loss** during regression. Plus, it uses a special pruning technique.

##### CART Pruning
Recall the loss function of a tree $T$

$$
\begin{aligned}
C_{\alpha}(T) &= C(T) + \alpha|T| \\
\end{aligned}
$$

and we further define the loss of a node $C_\alpha(t)$ as the loss if we merge all of its children into itself (i.e. the loss of this node we would get if we end the tree there). In other words, for a subtree $T_t$ rooted at $t$, the loss before pruning is $C_\alpha (T)$ (smaller classification loss, larger punishment for model complexity) and the loss after pruning is $C\alpha(t)$ (larger classfication loss, almost no punishment for model complexity). Therefore, if $C_a(t) > C_a(T)$, meaning we'd rather not add child nodes and make the prediction right at the node $t$, we will decide to prune. The **alpha threshold** is the value of $\alpha$, for which we do not care whether we prune or not (as they result in the same amount of loss). The threshold $\alpha$ satisfies 

$$
\begin{aligned}
C_\alpha (t) &= C_\alpha (T_t) \\
C(t) + \alpha * 1 &= C(T_t) + \alpha |T_t| \\
\alpha &= \frac{C(t) - C(T_t)}{|T_t| - 1}

\end{aligned}
$$

Not surprisingly, when $\alpha$ is small enough, we will never choose to prune. As we slowly increase $\alpha$, we will find more and more 'prunable' nodes. We increase the $\alpha$, get new pruned trees by pruning all 'prunable' nodes, and save them separately. In the end, we select the best one of the pruned trees using cross validation.

The pseudo code is the following:

```python
def get_all_pruned_trees(tree):

all_trees = []
alpha = +inf
curr_tree = deep_copy(tree)
while curr_tree has more than 2 levels:
    # Get the minimum alpha value in current tree
    for node in traverse(curr_tree):
        alpha = min(alpha, getAlphaThreshold(node))

    # Prune the nodes with the min alpha values
    # function `prune` should modify `curr_tree`
    for node in traverse(curr_tree):
        if (getAlphaThreshold(node) == alpha):
            prune(node)

    # Save the tree for as a candidate for cross validation
    all_treee.append(deep_copy(curr_tree))
```


