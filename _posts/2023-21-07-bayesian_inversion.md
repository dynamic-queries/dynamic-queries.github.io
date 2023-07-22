---
layout: post
title: Bayesian Waveform Inversion
date: 2023-07-21 08:00:00-0000
description: Full waveform inversion with MCMC, HMC, NUTS
tags: InverseProblems
categories: projects
related_posts: false
urlcolor: "blue"
---

This is original work done as a part of a student research project during my masters studies at TUM with [Felix Dietrich](https://fd-research.com/). 

*All ideas, figures, equations, examples and code, unless cited are a work of my own.*

### Introduction

**Waveform inversion** is principally parameter estimation with three insights.

1. The parameter is a field over some domain $$\Omega$$.
2. Only partial observations of the quantity of interest are available.
3. Observables are obtained from evaluating a family of wave propagation models. 

Consider a simple example of an acoustic wave equation.

$$
\begin{align}
    x \in \Omega \subset \mathbb{R} \\ 
    t \in \mathbb{R}^+ \\ 
    \frac{\partial}{\partial t}  u(x,t) = \frac{\partial^2}{\partial x^2} \left[\frac{1}{c(x)^2} u(x,t)\right] \\ 
    u(x,0) = f(x) \\ 
    \frac{\partial }{\partial t}u(x,0) = g(x) \\
    u(y,t) = 0 \quad \forall y \in \delta \Omega
\end{align}
$$

Here $$f,g$$ are suitably chosen initial conditions that would sustain elastic progation of a wave in $$\Omega$$. In the context of waveform inversion, $$c(x)$$ is the parameter and $$u(z,t) \: \forall z \in \hat{\Omega} \subset \Omega$$ is the observable. For the sake of this article, consider the following example for $$c,u$$.

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
    <img style="border:1px solid black;" src="/assets/bayesian/parameter.svg" alt="spline-sim" style="width:100%">
  </div>
  <div class="column">
    <img style="border:1px solid black;" src="/assets/bayesian/observable.svg" alt="spline-sur" style="width:100%">
  </div>
</div> 

The goal is to reconstruct $$c(x)$$ (*to the left*) from $$u(z,t)$$ (*to the right*) measured at $$z=z_0$$ where $$z_0 \in \Omega$$. The optimal $$c^{*}(x)$$ is the minimizer to 

$$
\begin{align}
  \arg \min_{\hat{c} \in \mathcal{H}} \| \hat{u}(z_0,t; \hat{c}(x)) - u(z_0,t)\|
\end{align}
$$

where $$\hat{u}(x,t)$$ is evaluated by solving 