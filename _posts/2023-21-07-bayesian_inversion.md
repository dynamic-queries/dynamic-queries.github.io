---
layout: post
title: (Original Research) Bayesian Waveform Inversion
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
    \frac{\partial}{\partial t}  u(x,t) = \frac{\partial^2}{\partial x^2} \left[c(x)^2 u(x,t)\right] \\ 
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

In the presented cases, $$l_2$$ Tikhonov regularization is used. That is

$$
\begin{align}
  \arg \min_{a \in \mathcal{A}} \sum_{i}\| \hat{u}(z_i,t; \hat{c}(x;a)) - u(z_i,t)\| + \lambda \| \hat{c}(x;a) \|
\end{align}
$$


The values of the regularization constants and the number of iterations used for each of these algorithms is indicated below.

##### ADAM


This is a first order optimization algorithm introduced in 2014 by [Kingma](https://arxiv.org/abs/1412.6980). It is a variant of momentum based methods such as Adagrad and Nesterov optimization methods. Generically, if $$f_{\theta}$$ is the objective function that needs to be optimized with respect to the parameters $$\theta$$, it is done so as follows.

--- 
***Algorithm***

Intitialize:
1. Constants $$\beta_1 = 0.9, \beta_2 = 0.999, \alpha = 0.001 , \epsilon=10^{-8}$$
2. Initial positions and velocities $$m_0 = 0, v_0 = 0$$
3. Make a "good initial guess" $$\theta_0$$


For the $$t^{th}$$ iteration:

1. Evaluate the gradient of the function $$g_t := \nabla_{\theta} f_{\theta_{t-1}}$$
2. Compute "position" term : $$m_t = \frac{\beta_1}{1 - \beta_1 ^ t} m_{t-1} + \frac{1-\beta_1}{1 - \beta_1 ^ t} g_t$$
3. Compute "momentum" term : $$v_t = \frac{\beta_2}{1 - \beta_2 ^ t} v_{t-1} + \frac{1-\beta_2}{1 - \beta_2 ^ t} g_t^2$$
4. Update parameters : $$\theta_t = \theta_{t-1} - \alpha \frac{m_t}{\sqrt{v_t} + \epsilon}$$
   

For $$t \in [1,\textrm{maxiters}] \subset \mathbb{N}$$

1. Repeat the four previous steps.

---


Using the previous algorithm and with a regularization constant $$ \lambda = 2.01 \pm 10^{-4} $$, one obtains the following reconstruction.

<img style="border:1px solid black;" src="/assets/bayesian/recon_parameter_adam.svg" alt="spline-sur" style="width:60%">

##### BFGS

One can also use, higher order optimization methods that make use of  $$ \nabla_{\theta}(\nabla_{\theta} f_{\theta})$$. Alas, computing the Hessian for every iteration is computationally expensive.
[BFGS method](https://en.wikipedia.org/wiki/Broyden%E2%80%93Fletcher%E2%80%93Goldfarb%E2%80%93Shanno_algorithm) is a quasi-Netwon method which instead of computing the entire Hessian, approximates it with several evalutions of the gradient of $$f_{\theta}$$.

Akin, to the previous section, one obtains a minimizer $$\theta^*$$ for $$f_{\theta}$$ as follows.

--- 
***Algorithm***

Initialize:

1. Initial Hessian approximant and initial learning rate : $$B_0 = \mathcal{I}, \alpha_0 = 10^{-2}$$
2. Make a "good initial guess" $$\theta_0$$


For every $$k^{th}$$ iteration:

1. Estimate new search direction, $$p_k$$ by solving $$B_{k-1} p_k = -\nabla_{\theta} f_{\theta_{k-1}}$$
2. Estimate step size, $$\alpha_k$$ by solving $$\arg \min_{\alpha_k} f_{\theta}(\theta_{k-1} + \alpha_k p_k)$$
3. Compute new direction $$\theta_{k} = \theta_{k-1} + \alpha_k p_k$$
4. Estimate update vector $$y_k = \nabla_{\theta} f(\theta_{k}) - \nabla_{\theta} f(\theta_{k-1})$$
5. Let $$s_k = \alpha_k p_k$$
6. Update Hessian approximant $$B_{k} = B_{k-1} + \frac{y_k y_k^T}{y_k^T s_k} + \frac{B_{k-1} s_k^T s_k B_{k-1}^T}{s_k^T B_{k-1} s_k}$$


For $$k \in [1,\textrm{maxiters}] \subset \mathbb{N}$$

1. Repeat the six previous steps.


---


Using the previous algorithm and with a regularization constant $$ \lambda = 0.399 \pm 10^{-3} $$, one obtains the following reconstruction.

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
Plagued by the ill-posedness of frequentist estimation, it seems appropriate to turn to Bayesian estimation. Granted, one does need a good prior for Bayesian methods to make computational sense, however Bayesian inverse problems are well posed (As highlighted in [Stuart](https://www.cambridge.org/core/journals/acta-numerica/article/abs/inverse-problems-a-bayesian-perspective/587A3A0D480A1A7C2B1B284BCEDF7E23)). 

Appropriately, the Bayes rule for this inference problem can be written as

$$
\begin{align}
  p(a | u(z)) = \frac{p(u(z)|a)\:p(a)}{p(u(z))}
\end{align}
$$

It is known that, esimating the marginal density $$p(u(z))$$ is computationlly intractable. Therefore, in what follows two classes of Bayesian state estimation methods -- MCMC and HMC (and its variants) are used to estimate $$
p(a|u(z))
$$ from which one can compute the expectation, variance and other higher order moments of $$\hat{c}(x;a)$$.

##### Markov Chain Monte carlo

###### Metropolis Hasting sampling

###### Gibbs sampling

##### Hamiltonian Monte carlo

###### Plain HMC

###### No U-turn sampling HMC

### Code 

These results can be reproduced with the following code. *(Documentation coming soon.)*

<script src="https://gist.github.com/dynamic-queries/3473ab7b87744cd1ad07cddd74eabd03.js"></script>