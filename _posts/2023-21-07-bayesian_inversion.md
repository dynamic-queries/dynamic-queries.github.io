---
layout: post
title: (Original work) Bayesian Waveform Inversion
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

The goal is to reconstruct $$c(x)$$ (*to the left*) from $$u(z,t)$$ (*to the right*) measured at $$z=\{z_0, z_i\}$$ where $$z_i \in \Omega$$. The optimal $$c^{*}(x)$$ is the minimizer to 

$$
\begin{align}
  \label{eq:opt}
  \arg \min_{\hat{c} \in \mathcal{H}} \sum_{i}\| \hat{u}(z_i,t; \hat{c}(x)) - u(z_i,t)\|
\end{align}
$$

where $$\hat{u}(z_0,t)$$ is evaluated by solving the acoustic wave equation using a robust finite difference method *(see code)*. 

### Parameter estimation
Estimating $$c(x)$$ is non-trivial. Firstly, one has to choose an appropriate bases to parameterize this function. This of course varies on a case by case basis. For the example above, radial bases functions are good approximants. Therefore,

$$
\begin{align}
  c(x) = \sum_{i=1}^{N} a_i \phi_i (x) \\ 
  \phi_i(x) := \phi(x;x_i,\mu) = \exp\left[{-\left(\frac{x-x_0}{\mu}\right)^2}\right] \\ 
  N = 5
\end{align}
$$

Therefore the parameterization space is now $$\mathcal{A} \ni a$$. 

$$N$$ is, by design, chosen sufficiently small to faciliate comparison with Bayesian optimization.

#### Frequentist estimation
Problem (\ref{eq:opt}) is ill-posed in the sense of Hadamard. As a result, it is initially solved with tradional gradient based optimization algorithms with suitable regularization. The algorithms used in this pursuit, are briefy outlined and the "best possible" reconstructions are presented.

The reconstructions here, must be taken with a grain of salt, for it is entirely possible to come up with an excellent initial guess that could converge to the actual $$c(x)$$ accurately.

Furthermore, it is observed that the reconstructions are sensitive to the choice of the type of regularization and the magnitude of the regularization constant. Common regularizations to an $$l_2$$ loss includes weighted norms of inputs, finite difference of $$l_2$$ loss with respect to time, fourier coefficients of $$l_2$$ loss and so on. 

In the presented cases, $$l_2$$ Tikhonov regularization is used. The values of the regularization constants and the number of iterations used for each of these algorithms is indicated below.

##### ADAM

--- 
*Algorithm*

---

Regularization constant $$ \lambda = 2.01 \pm 10^{-4} $$

<img style="border:1px solid black;" src="/assets/bayesian/recon_parameter_adam.svg" alt="spline-sur" style="width:60%">

##### BFGS

--- 
*Algorithm*

---

Regularization constant $$ \lambda = 0.399 \pm 10^{-3} $$

<img style="border:1px solid black;" src="/assets/bayesian/recon_parameter_bfgs.svg" alt="spline-sur" style="width:60%">

##### Issues

There are two difficulties that one encounters with the frequentist estimation while treating an ill-conditioned problem such as this one. 

1. Good initial guesses are vital for fast convergence.
2. Estimation is very sensitive to the value of $$\lambda$$.
   
Elaborating on (2), consider the following reconstructions with BFGS optimization for $$\lambda = 0.397,\:0.399\:,0.40\:,0.402$$  respectively.

<div class="row">
  <div class="column">
    <img style="border:1px solid black;" src="/assets/bayesian/bfgs_0.397.svg" alt="spline-sim" style="width:80%">
  </div>
  <div class="column">
    <img style="border:1px solid black;" src="/assets/bayesian/bfgs_0.399.svg" alt="spline-sim" style="width:80%">
  </div>
  <div class="column">
    <img style="border:1px solid black;" src="/assets/bayesian/bfgs_0.4.svg" alt="spline-sur" style="width:80%">
  </div>
  <div class="column">
    <img style="border:1px solid black;" src="/assets/bayesian/bfgs_0.402.svg" alt="spline-sur" style="width:80%">
  </div>
</div> 

Such sensitivity makes any practical grid search (which is performed at a much higher resolution) useless, questioning the practicality of such optimization routines for practical applications.

#### Bayesian estimation
Plagued by the ill-posedness of frequentist estimation, it seems appropriate to turn to Bayesian estimation. Granted, one does need a good prior for Bayesian methods to make computational sense, however Bayesian inverse problems are well posed (As highlighted in [Stuart](https://www.cambridge.org/core/journals/acta-numerica/article/abs/inverse-problems-a-bayesian-perspective/587A3A0D480A1A7C2B1B284BCEDF7E23)). In what follows, two classes of Bayesian state estimation methods -- MCMC and HMC are used to estimate $$c(x)$$. Relative merits and pitfalls, in contrast with frequentist methods are discussed.


##### Markov Chain Monte carlo

##### Hamiltonian Monte carlo

### Code 

These results can be reproduced with the following code. *(Documentation coming soon.)*

<script src="https://gist.github.com/dynamic-queries/3473ab7b87744cd1ad07cddd74eabd03.js"></script>