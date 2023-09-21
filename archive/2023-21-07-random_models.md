---
layout: post
title: (Original Research) Random feature potentials
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
Quantum mechanics is the mathematical framework used to describe physical systems. As it is the case with any universal theory, there are different formulations and interpretations of quantum mechanics. Three such formalisms that have stood the test of time  -- *the wavefunction formalism, the density matrix formalism and the Wigner function formalism.* For the rest of the article, I will concern myself with the wavefunction formalism of quantum mechanics.

Consider a physical system (molecular system) $$\mathcal{S}$$. In quantum mechanics, the state of the systems is completely described by the wavefunction -- $$
|\Psi\rangle : R \times s  \mapsto \mathbb{C}
$$. Here $$R,s$$ are the internal coordinates and spins of the constituents of $$\mathcal{S}$$. Generally, if each coordinate $$\{(R_i,s_i): i \in [1,N]\}$$ can assume $$d$$ different values, then $$
|\Psi\rangle
$$ maps a $$d^N$$ dimensional input vector space to $$\mathbb{C}$$. In some sense, the wavefunction can be categorized as a high dimensional function. Representing the wavefunction and performing operations on it suffers from what Bellman referred to as the curse of dimensionality.

Traditionally, the Hamiltonian $$\mathcal{H}$$ is an operator that acts on $$
|\Psi\rangle
$$. While one is interested in several interesting mutations of this operation, one that is particularly useful is determining the lowest eigenvalue of $$\mathcal{H}$$; as this directly corresponds to the ground state energy of $$\mathcal{S}$$. (Physical systems -- macroscopic and microscopic like to occupy the lowest energy level at equilibirum.) To wit,

$$
\begin{align}
  \mathcal{H} |\chi_0\rangle = \mathcal{E}_0 |\chi_0\rangle
\end{align}
$$

$$\chi_0$$ is the lowest energy state of $$\mathcal{S}$$ and $$\mathcal{E}_0$$ is its energy. In quantum chemistry $$\mathcal{E}_0 : R \times s \mapsto \mathbb{R}$$ is referred to as the **potential energy surface**. $$\mathcal{E}_0$$ is again a high dimensional function. 

One can in principle approximate it, by diagonalizing $$\mathcal{H}$$. Diagonalizing a discretized version of $$\mathcal{H}$$, say $$\mathcal{H}_d$$ results in a matrix. $$\mathcal{H}_d \in \mathbb{C}^{d^N \times d^N}$$. Using a dense diagonalization routine is computationally of the order $$\mathcal{O}(d^{3N})$$. Iterative Ritz minimization inturn is atleast $$\mathcal{O}(d^{2N})$$ with pathalogical scaling constants. Considering that $$\mathcal{H}_d$$ is Hermitian, it can also be efficiently digonalized using an Arnoldi iteration. But the truth remains -- exact diagonalization methods work well only for small $$N$$.

An efficient, widely prevalent, rather unintuitive method for determining $$\mathcal{E}_0$$ is using Density functional theory. Its based on Hohenberg Kohn's theorem -- *stating that one does not need knowledge of the wavefunction of a system to determine its ground state*. This directly excludes operating on a high dimensional Hilbert space but instead allows us to work with tractable mathematical quantities -- the density of states $$\rho$$.



### Appendix

##### Arnoldi's iteration


##### Ritz's minimization


##### Dense diagonalization -- QR decomposition