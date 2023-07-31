---
layout: post
title: Learning Hamiltonians with GP
date: 2023-07-20 08:00:00-0000
description: Inferring invariants of dynamical systems with gaussian processes.
tags: SystemsIdentification
categories: projects
related_posts: false
urlcolor: "blue"
---

This work is a re-creation of a method for learning the Hamiltonian of a dynamical system from [Bertlan and Dietrich](https://pubs.aip.org/aip/cha/article/29/12/121107/1027304/On-learning-Hamiltonian-systems-from-data).

*All figures, equations, examples and code are a work of my own. Only the idea has been borrowed from the paper.*

### Introduction

Most physical systems lend themselves to description using Hamiltonian mechanics. Briefly put:

For an $$N$$ body system living in $$d$$ dimensions, the classical mechanical state of the system is described sufficiently using a pair of canonical coordinates $$q,p \in \mathbb{R}^{dN}$$. For an evolutionary system, the canonical coordinates depend on time. Evidently, such evolutions are governed by Newton's equations. However, Netwon's equation are bulky to compute, analyze for large $$N$$. This problem was recognized by Hamilton and subsequently led to what is now referred to the Hamiltonian formulation of classical mechanics. 

Hamilton postulated that there exists a function $$H : q \times p \mapsto \mathbb{R}$$ -- the Hamiltonian. The evolution equations in this framework were given by the following PDE:

$$
\begin{align}
    \label{eq:Ham}
    \frac{\partial}{\partial t} q(t) = \frac{\partial H}{\partial p}\\ 
    \frac{\partial}{\partial t} p(t) = -\frac{\partial H}{\partial q}
\end{align}
$$

Or more compactly

$$
     {\partial_t} z := {\partial_t}\begin{bmatrix} q \\ p \end{bmatrix} = \begin{bmatrix} \mathcal{0} &&  \mathcal{I} \\ -\mathcal{I} && \mathcal{0} \end{bmatrix} \partial_{\begin{bmatrix} q & p \end{bmatrix}^{T}} H(q,p)
$$

Given observations $$\left([q(t_1),q(t_2),...,q(t_m)],[p(t_1),p(t_2),...,p(t_m)]\right)$$, I am interested in learning $$H$$ using a Gaussian process.

### Method

Let $$ H(z) \sim \mathcal{GP(\mu_z,\kappa(z,z'))}$$. 

$$
\begin{align}
  \mathbb{E}(H(z^*)) = k(z^* ,z)\: k(z ,z')^{-1}\:H(z') \\
  \frac{\partial}{\partial z^*}\mathbb{E}(H(z^*)) = \mathcal{J}^{-1} \frac{\partial z^*}{\partial t} \\
  \left[\frac{\partial}{\partial z^*}k(z^* ,z)\right]\: \left[k(z ,z')\right]^{-1}\:H(z') =\mathcal{J}^{-1} \frac{\partial z^*}{\partial t} 
\end{align}
$$

Here, $$z^*$$ and $$\frac{\partial z^*}{\partial t}$$ are known reference points. $$z'$$ are the validation points where the derivative of the Hamiltonian is not known. In order to determine this, one proceeds according to the following algorithm.

---

***Algorithm***

Given 
1. Snapshots $$z^*(t_d) = \left[z^*(1), z^*(2) ... z^*(T) \right]$$
2. Query points $$z_1, z_2, ...,z_N$$
3. Covariance kernel $$\kappa$$

Do
1. Evaluate $$\frac{d z^*}{dt}$$ using any higher order finite difference scheme.
2. Setup $$b:=\mathcal{J}^{-1} \: \frac{d z^*}{dt}$$
3. Compute $$K_{zz'} \in \mathbb{R}^{N \times N}$$ where $$K_{zz'}^{i,j} := \kappa(z^i,z^j)$$
4. Invert $$K_{zz'}$$
5. Compute $$M \in \mathbb{R}^{2dN \times N}$$ where $$M^{:,j} := \frac{\partial}{\partial z^*} \kappa(z^*,z)$$
6. Evaluate inference matrix $$I := M K_{zz'}^{-1}$$
7. Solve $$I H_{z'} = b$$ for $$H_{z'}$$


---

### Examples

##### Simple non-linear pendulum

A pendulum is a prototypical example of a Hamiltonian dynamical system. As shown below it is a one-dimensional dynamical system, described by the amplitude of oscillations $$\theta$$ and its time derivative $$\dot{\theta}$$.

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

<style>
.center {
  display: block;
  margin-left: auto;
  margin-right: auto;
  width: 50%;
}
</style>

<img style="border:1px solid black;" class="center" src="/assets/hamiltonian/pendulum.svg" alt="spline-sur" style="width:15%">


The equations of motion of a pendulum is the classical equation 
$$
\begin{align}
    \frac{d^2 \theta}{dt^2} = -\frac{g}{l} sin(\theta)   
\end{align}
$$
which when solved for an ensemble of initial conditions, results in its phase space. The phase space and its tangent bundle (its derivative) is shown below.
<div class="row">
  <div class="column">
    <img style="border:1px solid black;" src="/assets/hamiltonian/mini_sampled_trajectories.svg" alt="spline-sim" style="width:100%">
  </div>
  <div class="column">
    <img style="border:1px solid black;" src="/assets/hamiltonian/mini_sampled_velocities.svg" alt="spline-sur" style="width:100%">
  </div>
</div> 

Note that all the points in the images above satisfies the PDE in (\ref{eq:Ham}). Therefore, one could uniformly draw samples from this space to learn its Hamiltonian. (*The figures above show 500 unique samples in red.*)
