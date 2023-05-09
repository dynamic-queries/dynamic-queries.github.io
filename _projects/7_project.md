---
layout: page
title: FWI
description: Full Waveform Inversion with Neural Operators.
img:
importance: 1
category: Uni-Math
---
---

## Introduction

Full waveform inversion (FWI) is an effective technique to determine the subsurface mechanical properties of geological masses and engineering structures. It differs from other tomographic techniques in making use of the entire wavefield -- the amplitude, phase and frequency, to estimate features with high resolution. It consists of minimizing the distance between a measured signal and an artifically simulated one. 

Formally, let $$x \in \Omega$$ be the domain of interest and let $$m_{o}(x)$$ be an observable defined over $$\Omega$$. Let $$A : m(x) \times x \mapsto u(x)$$ be an operator, mapping the observable and the domain to the wavefield $$u$$. ($$A$$ is also known as the forward map.) If $$u_{o}(x)$$ is the wavefield corresponding to $$m_{o}(x)$$, then FWI is principally a minimization problem defined as:

$$
\begin{equation}
    \label{eq:fwi}
    \arg \min_{m \in \mathcal{M}} ||u_{o}(x) - A(m(x),x)||_p
\end{equation}
$$
where $$\mathcal{M}$$ is the hypothesis space of candidate observables explored with respect to a suitable $$p$$ norm.


Since its first formulation in 1977, much has been discovered about FWI with respect to the efficacy of optimizers (SGD vs quasi-Newton vs Newton vs Truncated Newton) and the choice of measures ($$L_p$$ norm vs Wasserstein measure). Furthermore, advances in computing has also benefitted this discipline; with highly parallel solvers that operate on very large time and length scales. 

Recently, machine learning based surrogates for forward maps are gaining traction due to their computational efficacy. Surrogates based on Neural Operators in particular have recently shown much promise for several applications in literature. In this work, we demostrate waveform inversion with such surrogates and outline their applicability for highly expensive optimization frameworks based on Bayesian inversion or optimal control. 

---
## Problem Setup
Let $$\Omega \subset \mathbb{R}^2$$.

For a reference observable (velocity) $$m_0:\Omega \mapsto \mathbb{R}$$, let the waveform be $$u_0(x,t) = A(m_0(x),x) \:\: \forall x \in \Omega$$. 

Assuming that from an initial measurement (oracle), $$u_0(x_s,t) \:\: \forall x_s \subset \Omega$$, is given, we are interested in numerically estimating $$m_0(x)$$.

---
For instance, one could imagine the reference velocity $$m_0(x)$$ as being : 
{% include figure.html path="assets/FWI/reference_defect.png" class="img-fluid" %} 

and the corresponding waveform sampled at seven different locations (sensors) in $$\Omega$$ as
{% include figure.html path="assets/FWI/reference_waveform.png" class="img-fluid" %} 

--- 

The optimization problem in $$(\ref{eq:fwi})$$ minimizes the distance betweem the above waveform (say) $$u_{m_0}$$ and a waveform $$u$$ obtained from a guessed observable $$m(x)$$. Applications such as this, which repeatedly invoke an expensive computational routine such as the waveform map $$A$$, are referred to as outerloop applications. 

Depending upon the choice of basis used in discretization, computational routines involving $$A$$ can be very expensive, usually due to the curse of dimensionality. Alternatively, we use Neural Operator surrogates for approximating $$A$$.