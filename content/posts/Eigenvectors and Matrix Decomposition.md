---
title: "Eigenvectors and Matrix Decomposition"
date: 2018-06-16T16:41:57+08:00
draft: false
---

#  Basic concepts
## Linear Transformation
A matrix can be seen as a linear transforamtion, for which the most important factors are the speed and direction:
- Eigenvalue is the velocity
- Eigenvector is the direction


## Rank
The rank of the matrix represents the dimension. It also indicates the number of the eigenvectors (linear independent base vectors) of the transformation.

---

# Eigenvectors and Eigenvalue

Matrix $A$ is a linear transformation, and it can be represents as follows:

$$Au = \lambda u$$

$u$ is one of the eigenvectors of matrix $A$. The equation shows that the linear transformation of $A$ only scales the eigenvector $u$ by $\lambda$ times. 

Eigenvectors can be seen as base vectors in a coordinate system during the linear transformation. The linear transformation equals to a scaling operation on the direction of eigenvectors where the scaling factors are eigenvalues.

---

# Feature Decomposition
If a $n\times n$ matrix $A$ has $n$ eigenvectors (the rank of $A$ equals to $n$), then matrix $A$ are decomposable.
$$A = Q^{-1}\Lambda Q$$
$Q$ is a square matrix of $n$ eigenvectors, where the $i^{th}$ column is the eigenvector $e_i$.
$\Lambda$ is a diagonal matrix where values on the diagonal are eigenvales (scaling values).