---
layout: post
title: (Original work) Random feature potentials
date: 2023-07-22 08:00:00-0000
description: Inferring potential energy surfaces with random feature models.
tags: Surrogatization
categories: projects
related_posts: false
urlcolor: "blue"
---

This is original work done as a part of my masters studies at TUM with [Chrisitian Mendl](http://christian.mendl.net/) and [Felix Dietrich](https://fd-research.com/).

*All ideas, figures, equations, examples and code, unless cited, are a work of my own.*


### Introduction

The importance of understanding the composition and behaviour of molecules is well established. Scientific disciplines such as chemistry, molecular biology, drug discovery and others, rely on molecular dynamics simulations as their core computational tool. 
Tradionally, MD has been and to some extent still is, dependent on empirical force fields for estimating the interaction forces among molecules. 

These empirical models are a simplification at best. The "proper" way to evaluate the forces in a molecular dynamics simulation is to use first principle calculations that solves the stationary Schrodinger's equations. But self-consistent loop sub-routines in most first principle calculations are too expensive to simulate molecules realistically for time domains where phenomena of interest emerge. 

This brings us to the fore of research today, where machine learning algorithms (deep learning, especially) has been co-opted to "learn" the rules of quantum mechanics from huge clusters of molecular data, generated from first principle calculations. The models used are the usual suspects -- neural networks, gaussian processes and other kernel methods.

In this work, we investigate the efficacy of random feature models as suitable surrogates for learning the ground state potentials of molecular systems. 

*I am aware that these two topics are esoteric in their own right. Therefore, I provide a quick interlude that discusses the notions of 1) Potential energy surfaces of molecular systems and 2) Random feature models as function approximation tools.*

### Interlude


#### Potential energy surfaces

#### Random feature models
Consider a sufficiently regular function $$f:x \mapsto y$$. Then one can find bases functions $$\phi_i(x; \theta)$$ with parameteric dependence on $$\theta$$ such that

$$
\begin{align}
    f(x) \approx \sum_i a_i \phi_i(x; \theta)
\end{align}
$$

This is a common feature in most function approximation schemes. In particular, if one uses a neural network with one hidden layer:

$$
\begin{align}
    \label{eq:randomNN}
    f(x) \approx W_2 \: \rho \left(W_1 x  + b_1 \right)
\end{align}
$$

Tradionally, one randomly chooses the parameters which are subsequently trained with a gradient based algorithm. If one assumes that the weights $$\theta= \{W_1, W_2, b_1\}$$ follow a probability distribution $$p_{\theta}$$, during training this distribution is modified with every epoch.

Random feature models, on the otherhand,  sample $$\theta \sim \mathcal{U}_{\theta}$$ from a uniform distribution (which requires no training). As surprising it is, it has been shown with uniform sampling of $$\theta$$ in (\ref{eq:randomNN}), used in conjunction with $$\rho = \{cos, tanh, relu\}$$ is a universal approximator. Further, one can define a litany of heuristics to define ad-hoc "probability" functions $$p_i$$, which when used for sampling should yield more accurate approximations at the same cost. 

Shown below is the function $$x \mapsto sin(x)$$ trained with an ADAM optimizer and one sampled using the sampling algorithm (I will discuss this next.). The prediction from the sampling algorithm is one shot, requiring no iterative training training, that is characteristic of all gradient based training methods.

<style>
    .column {
  float: left;
  width: 50.00%;
  margin : 0 0 0px 0px;
  padding: 2px;
}

/* Clear floats after image containers */
.row::after {
  content: "";
  clear: both;
  display: table;
}
</style>

<div class="row">
  <div class="column">
    <img style="border:1px solid black;" src="/assets/random_feature/single_output.svg" alt="spline-sim" style="width:100%">
  </div>
  <div class="column">
    <img style="border:1px solid black;" src="/assets/random_feature/single_sampling.svg" alt="spline-sur" style="width:100%">
  </div>
</div> 


This is also the case for a multioutput function such as

$$x \mapsto \begin{bmatrix} sin(4x) \\ cos(4x) \\ sin(4x) \: cos(4x)\end{bmatrix}$$

<div class="row">
  <div class="column">
    <img style="border:1px solid black;" src="/assets/random_feature/multi_output.svg" alt="spline-sim" style="width:100%">
  </div>
  <div class="column">
    <img style="border:1px solid black;" src="/assets/random_feature/multi_sampling.svg" alt="spline-sur" style="width:100%">
  </div>
</div> 

### Code

##### Random feature model for approximating simple functions

<script src="https://gist.github.com/dynamic-queries/7fe5162d9f355fe9e7414cf50a8fdfa0.js"></script>