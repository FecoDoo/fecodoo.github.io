---
title: "Bayesian Analysis"
date: 2021-09-19T17:33:22+03:00
draft: false

---

## Terms

- Probability: The chance of the occurrence of an event
- Probability Mass: The chance of the occurrence of an **random discrete event**
- Probability Density: The probability that a continues event variable is exactly equal to some value  
- Probability Mass Function (PMF): A function which gives **the probability that a discrete random variable is exactly equal to some value**.
- Probability Density Function (PDF): A function which gives **the probability that a continues random variable is exactly equal to some value**.

## Predictive Distribution

Now with Bayesian Model, we can get the **prior/posterior probability distribution of an extra sample $\tilde y$**.

$$ P(\tilde y | y,n,M) = \int_{0}^{1} P(\tilde y | \theta, y, n, M) P(\theta | y, n, M) d\theta \\ = \int_{0}^{1} \theta P(\theta | y, n, M) d\theta \\ = E(\theta | y) $$

where $n$ represents the number of experiment that have been taken already, $y$ stands for the event, and $M$ denotes the prior model we assume.

With a `binomial prior` $M$, we can have:

$$E(\theta | y) = \frac{y+1}{n+2}$$

