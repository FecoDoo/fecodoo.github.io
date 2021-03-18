---
title: "Machine Learning for Small Molecule Identification"
date: 2021-03-15T21:49:08+08:00
draft: false
---

## 1 Abstract

In metabolite anaysis, we are trying to study and annotate different molecules by thier features. However, small molecules typically contains similar structures and the space of potencial structures is tremendous (over 600 million). The amout of unique spectra covers only a small fraction of the entire database.

With the input out of the LC-MS^2 process, the machine learning algorithm runs like a search engine which assign ranked scores to predicted structures of high similarities. The point is to find suitable features. The output from `LC-MS` is flat feature vectors, and one common approach of is kernel methods which map embedded structured data into non-linear feature spaces.

## 2 Preprocessing

### 2.1 LC-MS

#### 2.1.1 Liquid Chromatography

Liquid Chromatography methods . It consists of 2 phases: mobile phase and stationary phase. LC can seperate molecules by their physicalchemical interactions with phases. Molecules that have stronger interactions with the staionary phase tend to ramain longer within the system, while those show weaker interactions spend less `retension time` inside the system.

This process seperate molecules from `time (physicalchemical interaction) dimension`.

#### 2.1.2 Mass Spectrometry

Mass Spectrometry diffirenciates molecules according to their mass. This process ionize the molecules and provide a seperation from `mass dimension`.

#### 2.1.3 LC-MS^2 (Tandem Mass Spectrometry)

After a standard `LC-MS` workflow, we can add an additional MS process which break the target molecule into fragments, and then performs an analysis on the debris. It help us to understand the structure of the target molecule.

## 3 Kernel Methods

Kernel methods are used to map features into higher dimensions for better seperation.

### 3.1 Non-linear Feature Map

If $κ$ is kernel, than there exists a function $\phi:X \rightarrow F_X$ (feature map) such that:

$$κ(x,x′) = \langle\phi(x),\phi(x′)\rangle F_X$$

In practice we don't have to get what $φ$ really is, this is called "kernel trick". We can consider operation $κ$ as similarity measure between objects in $X$.

### 3.2 Kernelized SVM

A typical kernelized Support Vector Machine (SVM) for binary classification can be described as:

$$ h(x) = sign((\sum_{i=1}^{n}y_i\alpha_ik(x_i,x))+b) = sign(\langle w,\phi(x)\rangle + b) $$

- The training dataset is ${(x_i,y_i)}_{i=1}^{n}$
- $\alpha_i \in R$ is the dual variables, $b \in R$ is the bias term
- $κ(x_i,x)$ represents the similarity between training example $i$ and “new” example $x$
- Kernel trick application: $w=\sum_i y_i\alpha_i \phi(x_i) \in F_X$

#### 3.2.1 Model Optimization in the Dual Space

To search for the best $\alpha_i$, we need to:

$$\mathop{max}\limits_{\alpha} \sum_\limits{i=1}^{n}\alpha_i - \frac{1}{2}\sum_\limits{i=1}^{n}\sum_\limits{j=1}^{n} y_iy_j\alpha_i \alpha_j k(x_i,x_j) $$

$$s\.t\. 0\le \alpha_i \le C, \forall i \in {1,2,...,n};\sum_\limits{i=1}^{n}\alpha_iy_i=0$$

## 4 CSI: FingerID to Prediction