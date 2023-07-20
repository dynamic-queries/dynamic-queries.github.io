---
layout: page
title: Green learning
description: Learning Green's functions for non-homogenous differential equations.
img: 
importance: 2
category: 2023
---

### Introduction

Green's functions are fascinating mathematical objects. 

Consider
$$
\begin{align}
    x \in \Omega \subseteq \mathbb{R}^d \\ 
    y \in \Delta \subseteq \mathbb{R}^m \\ 
    z \in \Upsilon \subseteq \mathbb{R}^n \\ 
    \mathcal{F} \ni f:x \mapsto y \\
    \mathcal{G} \ni g:x \mapsto z \\
    \mathcal{L}: f \mapsto g
\end{align}
$$ 

If $$\mathcal{L}$$ is linear, then

$$
\begin{align}
    g(x) = \int_{\Omega} dy\:G(x,y) \: f(y)
\end{align}
$$

Intuitively, given a linear operator $$\mathcal{L}$$, the Green's function $$G_{\mathcal{L}}$$ prescribes a computational routine where, performing a specific bases transformation of the input function $$f$$ results in the output function $$g$$.

### Learning from data

Recently, learning dynamical systems from data is gaining popularity, largely due to the increasingly cheaper means to collect, store and transmit data. From these considerations, it is natural to ask, given a reference dataset $$D=\{(\hat{f}_i,\hat{g}_i): i \in \mathbb{N}\}$$ with parameterized measurements $$\hat{f}, \hat{g}$$, what is the Green's function corresponding to the dynamical system generating this data.

A trivial idea, that is also quite easy to compute is to parameterize $$G$$ using a neural network.
$$
\begin{equation}
    \label{eq:NN}
   g(x) = \int_{\Omega} dy\:G_{NN}(x,y) \: f(y) 
\end{equation}
$$

Depending on the quadrature scheme used to evalute this integral, there can be different formulations of (\ref{eq:NN}) (subtly represented as the quadrature operator $$\mathcal{Q}$$). In any case, this will always result in a linear transform:

$$
\begin{align}
    \label{eq:disc-NN}
    \hat{g}(x_i) = \mathcal{Q} [\mathcal{K_{x,y}} \: \mathcal{a} \circledcirc \hat{f}(y_j)] \\ 
    \mathcal{Q}^{\dagger} \hat{g}(x_i) =  [\mathcal{K_{x,y}} \: \mathcal{a} \circledcirc \hat{f}(y_j)] \\
    l := \frac{\mathcal{Q}^{\dagger} \hat{g}(x_i)}{\hat{f}(y_j)} = \mathcal{K_{x,y}} \: \mathcal{a}
\end{align}
$$

This needs some clarification. The inputs to the $$G_{NN}$$ are $$x_i, y_j$$. However, in the orginal inference problem, only $$\hat{f}(x_i)$$ and $$\hat{g}(y_j)$$ are available. In order to exploit, the linear representation of $$G_{NN}$$, one has to recast the inference problem as one that learns $$G_{NN}$$ with the following reference data $$D_r = \{([x_i, y_j],l_{i,j}) : i,j \in \mathbb{N}\}$$.


#### Quadrature operators
Assuming that, inferring $$a$$ using a least squares procedure is sufficiently robust, the stability of the new inference scheme depends largely on the stability of the numerical algorithm used to compute $$\mathcal{Q}^{\dagger}$$. Common choices include:

- Riemanian quadrature $$\mathcal{Q}_{R}$$
- Gaussian quadrature $$\mathcal{Q}_{G}$$
- Other weighted polynomial quadrature 

In what follows, the spectrum of the different integration schemes are examined.

[... insert ideas about spectra of the quadrature operators ... ]()


### Universal approximation for rational activation functions
[Boulle]() showed that rational activation functions are superior to their traditionally used counterparts, for the purpose of learning Green's functions. The sampling algorithm, used in linearizing $$G_{NN}$$, has been proven to be a universal approximator for ReLU and tanh functions alone. Here we extend this result to the case of rational activation functions, as defined in [Boulle]().


### Examples

#### Linear ordinary differential equations


#### Linear partial differential equations

##### Time independent case

###### Diffusion equation

###### Euler beam equation

##### Time dependent case

###### Stokes equation

###### Burger's equation

###### Acoustic Wave equation

###### Maxwell's equations

###### Schrodinger's equation

#### Non-linear differential equations
It is known that in an appropritely chosen feature space, there exist linear duals to non-linear operators. These linear operators are inturn inferred using kernel methods. Here, we propose learning the Green's functions of these globally linearized operators, paving the way for analyzing the properties of a non-linear operator, in a linearized setting. The problem of coming up with a suitable definition for the feature space remains open and must be considered on a case-wise basis.

###### Lorenz 69 attractor

###### Robertson's reaction network

###### Non-linear Schrodinger's equation

###### Allen Cahn equation

###### KdV equations

### Engineering Applications
In this section, we demonstrate our method is applicable to several outer-loop applications that are of interest to the engineering community.

##### Waveform inversion

##### Flow control

##### Uncertainity quantification for rate coefficients of a LASER model


### Conclusion, claims and expectations
An efficient training procedure for learning the Green's function for linear partial differential equations is presented. The proposed method is tested for several commonly occuring linear partial differential equations. Finally, the applicability of this method to three distinct outer-loop applications is demonstrated.

Echoing the claims made in [Boulle](), our method represents a comparitively more interpretable inference algorithm for learning linear PDEs. It is our hope, that more engineering applications seek to digress from the black-box attributes of Neural Operators and make use of more transparent tools, that are available at their disposal.