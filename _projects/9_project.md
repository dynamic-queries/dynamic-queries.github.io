---
layout: page
title: MLP 
description: Inferring Potential Energy Surfaces from Trajectories of MD simulations.
img: 
importance: 2
category: Uni-Math
---


## Introduction
Any physical system is characterized by a certain set of properties. It is these properties that define what that physical object is. Metals, for instance, are electrically and thermally conductive. Ceramics have extreme resitance to abrasion, electrical and thermal stresses. Biological tissues exhibit among other things properties such as regeneration. There are seemingly endless objects and endless properties that they posses.

It is all the more impressive that a finite set of elements and compositions thereof, constitue this rich tapestry of physical systems. Naturally, this question intrigued the ancients and continues to intrigue the modern day researcher. What we do know is, interactions among these fundamental building blocks are crucial to explaining these emergent phenomena. 

In that pursuit there is a whole branch of mathematics -- potential theory, devoted to finding a unifying framework for analyzing these potentials. There are countless physicists (theoretical and experimental), finding the **"right interaction potential"** for a specific system. And finally, there are computational scientists working towards effective computational procedures for calculating these potentials.

Until recently, a typical workflow for estimating the interaction potential for a system $$\mathcal{S}$$ would look like :
1. Experimental data acquisition and validation by an experimentalist.
2. Data analysis and theorization by physicists.
3. In-silico validation of the theory by a computational scientist.

In the age of machine learning, steps 2,3 are aggregated and serves as an **"automation"** of scientific discovery. While the results from ML-Potentials are certainly impressive, there are also unexplainable. "Unexaplainability" of machine learning models often has a negative connotation, and in some sense it is justified. But it also indicates that there exist more accurate models that could potentially be discovered using current mathematical frameworks. This work is a step in that direction while leveraging classical statistical models.

## Problem setup
Consider a system $$\mathcal{S}$$ with $$n$$ identical interacting particles that live in a $$d$$ dimensional phase space with positions $$q \in \mathbb{R}^d$$ and momenta $$p \in \mathbb{R}^d$$. The evolution of $$z:=[q \quad p]^T$$ follows Newton's laws.

$$
\begin{align}
    \label{eq:newtonslaws}
    \frac{d}{dt}p(t) = -\nabla_{q} U(q(t)) \\ 
    \frac{d}{dt}q(t) = p(t)
\end{align}
$$

It is often the case in experiments that, one can accurately record observables such as $$q$$ (and sometimes $$p$$). Given $$q \textrm{ or } p$$, we ask ourselves how accurately one can infer $$U$$? This is equivalent to asking, How accurately can one solve the following optimization problem, 

$$
\begin{align}
    \label{eq:opt_prob}
    \arg \min_{\phi \in \mathcal{H}} \left\|\frac{d}{dt}p(t) + \nabla_{q} \phi(q(t))\right\|_{\mu} 
\end{align}
$$

in a hypothesis space $$\mathcal{H}$$ of potential functions $$\phi$$ with respect to some measure $$\mu$$. 

Assuming that the chosen measure $$\mu$$ leads to a smooth loss landspace and our optimizer converges to the global minimum (say SGD), the accuracy of the inferred potential $$\phi$$, depends on the "goodness" of $$\mathcal{H}$$. (We will later clarify what goodness could mean.) Therefore the goal of this work is to find good hypothesis spaces for some commonly recurring interaction problems in computational chemistry. 

#### Physical Systems of interest

We investigate the following systems; ordered with respect to increasing computational complexity.

1. Gas in an infinite enclosure (homogenous / heterogenous)
2. Gas in a finite enclosure with fixed walls (homogenous / heterogenous)
3. Gas in a finite enclosure with one moving wall (homogenous / heterogenous)
4. Nucleation of water droplet
5. Solvation of Lysozyme in Water
6. Homogeneous nucleation of water and carbon dioxide in carrier gas

A collection of descriptions to these systems and their computational models can be found [here](https://github.com/dynamic-queries/MolecularDynamicsZoo).

## Inference models