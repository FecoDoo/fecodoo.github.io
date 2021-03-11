---
title: "HOFM"
date: 2021-03-12T00:40:22+08:00
draft: false
---
## Abstract

According to practical experience and medical treatment records, it shows that combinations of multiple drugs may positively effect the curing process regarding to certain disease. People are investigating about the effectiveness of the drug combinations with certian dosage against senarios when used respectively. However, the number of potencial combinations may grow exponentially when try to add more medicines into consideration, this refers as `combinatorial explosion`. Hence, we need to prioritize and reduce ranks for the search for combinations, whereas certrain machine learning methods can contribute. In this lecture, we focused on `ComboFM`, which is based on `higher-order factorization machines`.

## Feature interactions

For a prediction problem where features show dependencies with each other, traditional linear regression techniques are not suitable since it does not take simultaneous interaction into consideration. However, calculating higher order interactions is extremely expensive, as an quadratic  regression prediciton model $f: R^d \rightarrow R$ has $O(d^2)$ parameters to be estimated.

## Factorization machines (FM)

Factorization machines introduces a much more practical approach, where the polynomial regression model is replaced by a factorized form.

$$ \hat{y}FM(x) = \sum_iw_ix_i + \sum_{i^{'}>i}<p_i,p_{i^{'}}>x_ix_{i^{'}} $$

The interaction weight $w_{ij} \approx <p_j, p_{j^{'}}> = \sum_{S=1}^{k}p_{j S}p_{j^{'} S}$ is represented as a inner product over the factor contributions, which refers as the factorization process.

## Higher-order factorization machines (HOFM)

The `HOFM` step further in decomposition as it canrepresent polynomial models of arbitrary maximum degree $m$. And the interaction weights are given by generalized inner products.

$$w_{j_1,j_2,...,j_m} = <p_{j_1}^m,...,p_{j_m}^m> = \sum_{S=1}^{k} p_{j_1S^m} ... p_{j_mS^m}$$

There will only be $O(dkm)$ parameters to estimate, compared to $O(dm)$ of the full mod.

### Objective function

The objective function of learning HOFMs is given as follows:

$$\frac{1}{n} \sum_{i=1}^{n} loss(y_i, \hat{y}HOFM(x_i)) + \frac{\beta_1}{2}||w||^2 + \frac{\beta_2}{2}||P||^2$$

The objective function shows that this is not a convex-optimizable, thus it's difficult to find global optimals. For non-convex optimization, we can use decent algorithms to search for local optimals.

In addition, we can notice that the calculation for weight $ w_{j_1,j_2,...,j_m} $ shares some part from the previous weight $ w_{j_1,j_2,...,j_{m-1}} $, so we can save a lot of time by deploying `dynamic programming` by storing results from each step.

## Problems with HOFM

In the case study, we assumed that `monotherapy responses`  is known in all scenarios. But if we add a new medicine to which we don't have sufficient information, it could be not easy to predict the effectiveness of new combinations. In a matrix view, the new $p_{new}$ is unknown, and we the feature interaciton weight $w_{j_{exist},...,j_{new}}$ is always $0$.
