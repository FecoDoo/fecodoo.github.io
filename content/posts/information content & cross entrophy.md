---
title: "Information Content & Cross Entrophy"
date: 2020-01-22T18:40:34+02:00
draft: false
---

## 1 Information Content

### 1.1 Defination

In information theory, the `information content,` self-information, surprisal, or Shannon information is a basic quantity derived from the probability of a particular event occurring from a random variable.

The `information content` tells how much information is given.

### 1.2 Function

The function of information content need to comply with both constraints:
$$f(x) = \sum_{1}^{i} I(p_i)$$
$$x = \prod_{1}^{i} p_i$$

As a result, the function can be described as:

$$I(x) = - log_{n}(x)$$

Where $n$ is a factor describing the number of possible events in the measurement system, normally $n=2$, the metric of the information content is **bit**.

## 2 Entrophy

Unlike informaiton content, entrophy measures the uncertainty of a system. Given a event $e$ with probability of $p$ to happen, the entrophy is calculated as:

$$E(e) = p \times -log_n(p) + (1-p) \times -log_n(1-p)$$

## 3 KL Divergence

KL divergence, also named relative entropy, describes the similarity between two distributions or systems, such as Poisson and Gaussian.

The KL divergence function can be described as:

$$$KL(P||Q) = \sum_{1}^{i} p_i \times (-log_n(q_i) - \sum_{1}^{i} p_i \times (-log_n(p_i)$$

According to [Gibbs' inequality (吉布斯不等式)](https://en.wikipedia.org/wiki/Gibbs%27_inequality):
$$\sum_{1}^{i} p_i \times (-log_n(p_i) \lt \sum_{1}^{i} p_i \times (-log_n(q_i)$$

Hence the KL divergence will be constantly positive. The smaller the KL divergence is, the higher similarity the two distributions have.

### 3.1 Cross Entrophy

We can notice the second part from the KL divergence function is the entrophy of distribution $Q$, where the first part calculates something much similar to the entrophy of distribution $Q$, but on the base of $P$. This is called `cross entrophy`, which is widely used in evaluating neural network models.

$$L(y_{true}||y_{pred}) = -\sum_{1}^{n} (y_{true} \times log_{2}(y_{pred}) + (1-y_{true}) \times log_{2}(1 - y_{pred}))$$
