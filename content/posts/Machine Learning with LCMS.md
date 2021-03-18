---
title: "Machine Learning With LCMS"
date: 2021-03-18T19:10:53+08:00
draft: false
---

# Learning Diary 2

## 1 Abstract

In metabolite analysis, we are trying to study and annotate different molecules by their features. However, small molecules typically contain similar structures and the space of potential structures is tremendous (over 600 million). The amount of unique spectra covers only a small fraction of the entire database.

With the input out of the LC-MS^2 process, the machine learning algorithm runs like a search engine that assigns ranked scores to predicted structures of high similarities. The point is to find suitable features. The output from LC-MS is flat feature vectors, and one common approach is kernel methods which map embedded structured data into non-linear feature spaces.

## 2 Preprocessing

### 2.1 LC-MS

#### 2.1.1 Liquid Chromatography

Liquid Chromatography consists of 2 phases: mobile phase and stationary phase. LC can separate molecules by their physicochemical interactions with phases. Molecules that have stronger interactions with the stationary phase tend to remain longer within the system, while those who show weaker interactions spend less retention time inside the system.

This process separates molecules from the time (physicochemical interaction) dimension.

#### 2.1.2 Mass Spectrometry

Mass Spectrometry differentiates molecules according to their mass. This process ionizes the molecules and provides a separation from the mass dimension.

#### 2.1.3 LC-MS^2 (Tandem Mass Spectrometry)

After a standard LC-MS workflow, we can add an MS process that breaks the target molecule into fragments, and then performs an analysis on the debris. It helps us to understand the structure of the target molecule.

## 3 Kernel Methods

Kernel methods are used to map features into higher dimensions for better separation.

### 3.1 Non-linear Feature Map

If $κ$ is kernel, than there exists a function $\phi:X \rightarrow F_X$ (feature map) such that:

$$κ(x,x′) =<\phi(x),\phi(x′)>F_X$$

In practice we don't have to know what $φ$ really is, this is called "kernel trick". We can consider operation $κ$ as similarity measure between objects in $X$.

### 3.2 Kernelized SVM

A typical kernelized Support Vector Machine (SVM) for binary classification can be described as:

$$h(x) = sign((\sum_{i=1}^{n}y_i\alpha_ik(x_i,x))+b) = sign(<w,\phi(x)> + b)$$

- The training dataset is ${(x_i,y_i)}_{i=1}^{n}$
- $\alpha_i \in R$ is the dual variables, $b \in R$ is the bias term
- $κ(x_i,x)$ represents the similarity between training example $i$ and “new” example $x$
- Kernel trick application: $w=\sum_i y_i\alpha_i \phi(x_i) \in F_X$

#### 3.2.1 Model Optimization in the Dual Space

To search for the best $\alpha_i$, we need to:

$$\mathop{max}\limits_{\alpha} \sum_\limits{i=1}^{n}\alpha_i - \frac{1}{2}\sum_\limits{i=1}^{n}\sum_\limits{j=1}^{n} y_iy_j\alpha_i \alpha_j k(x_i,x_j)$$

$$s.t. 0\le \alpha_i \le C, \forall i \in {1,2,...,n};\sum_\limits{i=1}^{n}\alpha_iy_i=0$$

## 4 CSI: FingerID to Prediction

The fingerprint is an indicator vector that describes the substructure of the molecule. By comparing with real sub-structure vectors in the database, we can rank the similarities between predicted sub-structure with candidates.

## 5 Additional Features

There are more features that can be considered into the prediction model, including `retension time`, `biological context`, `collision cross section` and `relationship between MS features`.

# Chapter Concluding

Scientists are now using technologies such as Liquid Chromatography and Mass Spectrometry to measure sub-structures of biological molecules. By applying energy to break up molecules into fractions, we can obtain peaks on a mass spectrometry graph that indicate different physicochemical characteristics. The Liquid Chromatography process separates molecules from the temporal (physicochemical interaction) dimension, as different compounds interact with the LC system with varying intensities. The mass and interaction intensities combine to form a mass spectrogram of each component, called a fingerprint, which is used as a label for identification.

However, searching for sub-structures of the target molecule based on the database of known chemical molecules is computationally intensive and inefficient. The high degree of similarity between mass spectra makes it more difficult to distinguish between structurally similar molecules. Kernel methods provide an alternative way to enhance the searching process. We can now map features (ironized peaks) into higher dimensions for better separations. The kernel trick allows us to skip the calculation of the real mapping step and instead  focus on the distance calculating, which takes considerably less time.

The classifier usally rank series of potential sub-structures regarding to the target molecule with probabiity scores. The workflow of typicall model training based on FingerID and non-linear kerneled SVM includes 5 steps:

1. Process each compound from spectral database of know references
2. Compute fragmentation trees
3. Compute all-vs-all kernels
4. Compute molecular properties from molecular structure (normally a sub-structure indicating vector, known as fingerprint)
5. For each molecular property, train individual SVM using multiple kernel learning.

And given a new spectrum, the process is similar to the training process while the output step will be an estimation of Platt probabilities of potential sub-structures.

