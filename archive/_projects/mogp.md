---
layout: page
title: MOGP
description: Multi-Output Gaussian Processes.
img:
importance: 1
category: 2022
---

### Introduction
Gaussian processes are a popular **Bayesian** **learning** framework.

A typical *learning problem* consists of approximating an approximation $$\hat{f}$$ to the function $$f:x \mapsto y \quad \forall x,y \in \Omega \subset \mathbb{R}^d$$, given evaluations of the function at certain input points, that is $$\mathcal{D} = \{(x_i, f_i) : i \in \mathbb{N}\}$$.

*Bayesian inference* assumes that the input $$x$$ and the outputs $$y$$ are random variables with some densities $$P_x, P_y$$ respectively. These densities are related by

$$
    P(x,y) = P(y | x) P(x) = P(x | y) P(y)
$$

Note that we do not have a closed form for any of these probability functions and estimating probability densities from data is in itself a hard problem. Instead, we make guesses about what these functions might be. Historically, each of these probability functions have their own names, namely

1. $$P(y)$$ -- Prior
2. $$
    P(x | y)
    $$ -- Likelihood
3. $$P(x)$$ -- Marginal 
4. $$
    P(y | x)
    $$ -- Posterior 

Commonly, one assumes a functional form for the prior. The posterior is the updated version of the prior, once we have seen the reference data.

### Single Output GP

Further, samples $$y,x$$ are assumed to be related via 

$$y = f(x)$$

$$f:x \mapsto y \quad \forall x,y \in \Omega$$

Here $$f$$ is assumed to be any non-linear function. One can project $$x$$ onto a higher dimensional space spanned by the bases generated from a kernel function $$\kappa$$, also known as the **kernel trick**, where the non-linearity of $$f$$ is transformed into a linearity. Without any loss of generality, we can assume that any non-linear function can be transformed into a linear map.

$$y = Wx + b$$

$$W \in \mathbb{R}^{d \times d} \: \textrm{ and } b \in \mathbb{R}^d$$

With this parameteric form for the output, defining a prior for the weights, $$z := \begin{bmatrix} W & b \end{bmatrix}$$ is equivalent to defining a prior for the output, $$y$$. Therefore in a single output GP, there are three random variables $$x,y,z$$. Their joint probability distribution is given as 

$$
\begin{align}
    P(x,y,z) = P(x,y|z) P(z) = P(z|x,y) P(x,y) \\
 P(z|x,y) = \frac{P(x,y|z) P(z)}{P(x,y)} \\ 
 = \frac{P(y|x,z) P(x,z)}{P(x,y)} \\
  = \frac{P(y|x,z) P(x)P(z)}{P(y|x)P(x)}\\
 P(z|x,y) \propto P(y|x,z) P(z)
 \end{align}
$$

Here, $$z \sim \mathcal{N}(0, \Sigma_z)$$, that is

$$
\begin{align}
    P(z) = \frac{1}{\sqrt{(2\pi)^d |\Sigma_z|}} \exp\left(-\frac{1}{2} z^T \Sigma_z^{-1} z\right)
\end{align}
$$

Further, the likelihood is defined with the assumption that each component of $$x$$ is independent of one another. This is why this inference procedure is called single output Gaussian process. That is

$$
\begin{align}
    P(y|x,z) = \prod_{i=1}^{d} \frac{1}{\sqrt{2\pi \sigma^2}} \exp\left(-\frac{1}{2}\left(\frac{y_i - (W_i x_i + b_i)}{\sigma}\right)^2\right) \\ 
    P(y|x,z) = \frac{1}{(\sqrt{2\pi \sigma^2})^d} \exp\left(-\frac{1}{2\sigma^2}\sum_{i=1}^{d} (y_i - (W_i x_i + b_i))^2\right) \\ 
    = \frac{1}{\sqrt{(2\pi)^d |\Sigma_y|}} \exp\left(-\frac{1}{2}(y - (W x + b))^T \Sigma_y^{-1} (y - (W x + b))\right)
\end{align}
$$

From this $$
P(y|x,z)
$$ is a multivariate normal distribution $$\mathcal{N}(Wx+b, \Sigma_y)$$. 

Further one can write down the posterior as 

$$
\begin{align}
    P(z|x,y) \propto P(y|x,z) P(z) \\ 
    = \frac{1}{(2\pi)^d \sqrt{|\Sigma_z||\Sigma_y|}} \exp\left(-\frac{1}{2} \left[z^T \Sigma_z^{-1} z + (y - (W x + b))^T \Sigma_y^{-1} (y - (W x + b)) \right]\right) \\ 
    = \frac{1}{(2\pi)^d \sqrt{|\Sigma_z||\Sigma_y|}} \exp\left(-\frac{1}{2} \left[z^T \Sigma_z^{-1} z + (y - z x')^T \Sigma_y^{-1} (y - z x') \right]\right)
\end{align}
$$