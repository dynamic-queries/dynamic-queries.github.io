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

Let $$ H \sim \mathcal{GP(\mu_z,\kappa(z,z'))}$$. 