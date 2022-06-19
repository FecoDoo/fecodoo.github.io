---
title: "PageRank & SimRank"
date: 2020-01-13T22:31:23+08:00
draft: false
---

## Random Walk

Given a graph, a random walk is an iterative process that starts from a random vertex, and at each step, either follows a random outgoing edge of the current vertex or jumps to a random vertex. The jump part is important because some vertices may not have any outgoing edges so a walk will terminate at those places without jumping to another vertex.

## Page Rank

(PR) measures **stationary distribution of one specific kind of random walk that starts from a random vertex and in each iteration, with a predefined probability *p*, jumps to a random vertex, and with probability*1-p* follows a random outgoing edge of the current vertex.** Page rank is usually conducted on a graph with homogeneous edges, for example, a graph with edges in the form of "A linksTo B", "A references B", or "A likes B", or "A endorses B", or "A readsBlogsWrittenBy B", or "A hasImpactOn B".

## Personalized Page Rank

(PPR) is the same as PR other than the fact that **jumps are back to one of a given set of starting vertices**. In a way, the walk in PPR is biased towards (or personalized for) this set of starting vertices and is more localized compared to the random walk performed in PR.

## Page Rank Matrix

https://zhuanlan.zhihu.com/p/137561088

# Citation

[Intuitive Explanation of Personalized Page Rank and its Application in Recommendation - Alan Wu](https://blogs.oracle.com/bigdataspatialgraph/intuitive-explanation-of-personalized-page-rank-and-its-application-in-recommendation)