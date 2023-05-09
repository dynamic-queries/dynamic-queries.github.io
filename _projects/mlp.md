---
layout: page
title: MLP 
description: Inferring Potential Energy Surfaces from Trajectories of MD simulations.
img: 
importance: 2
category: 2023
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

#### First attempt

Consider the case where, $$n_{ics}=1$$ , $$n_{ts}=1$$ , $$n_p=100$$ , $$n_d=2$$ 

Recall, 
$$
\begin{equation}
    f_i:=\sum_{j=1}^{n_p}(q_i-q_j)\phi(\|q_i-q_j\|)    
\end{equation}
$$
where  $$q_i,q_j,f_i\in \mathbb{R}^2$$, $$\phi:\mathbb{R} \mapsto \mathbb{R}$$

Equivalently $$f_i=R_i \phi(r_i)$$ 
where $$R_i \in \mathbb{R}^{2 \times n_p}$$ and $$r_i \in \mathbb{R}^{n_p}$$ . 

Consider $$n_p=3$$ , 

$$
\begin{equation}
\begin{bmatrix} f_1 \\ f_2 \\ f_3 \end{bmatrix} = \begin{bmatrix} R_1 && && \\ && R_2 && \\ && && R_3\end{bmatrix} \begin{pmatrix}  \begin{bmatrix} phi(r_1) \\ phi(r_2) \\ phi(r_3) \end{bmatrix} \otimes \begin{bmatrix} 1 \\ 1\end{bmatrix}\end{pmatrix}
\end{equation}
$$

where $$R_1 \in \mathbb{R}^{2\times 2},R_2 \in \mathbb{R}^{1\times 1},R_3 \in \mathbb{R}^{0\times 0}$$  

and 

$$
\begin{equation}
\begin{bmatrix} phi(r_1) \\ phi(r_2) \\ phi(r_3) \end{bmatrix}=\begin{bmatrix}b_{11}&b_{12}&b_{13} \\ b_{21}&b_{22}&b_{23} \\ b_{31}&b_{32}&b_{33}\end{bmatrix} \begin{bmatrix} k_1 \\ k_2 \\ k_3 \end{bmatrix}
\end{equation}
$$


Then 
$$
\begin{equation}
\begin{bmatrix} f_1 \\ f_2 \\ f_3 \end{bmatrix} =\begin{bmatrix} R_1 & & \\ & R_2 & \\ & & R_3\end{bmatrix} \left(\begin{bmatrix} b_{11}&b_{12}&b_{13} \\ b_{21}&b_{22}&b_{23} \\ b_{31}&b_{32}&b_{33} \end{bmatrix}\otimes \begin{bmatrix} 1 &  \\ & 1 \end{bmatrix}\right)\left(\begin{bmatrix} k_1 \\ k_2 \\ k_3 \end{bmatrix} \otimes \begin{bmatrix} 1 \\ 1\end{bmatrix} \right)
\end{equation}
$$

This  inference procedure does not scale well.

#### Second attempt
A simple inference model for the potential assumes the following structure.

$$
\begin{equation}
    \label{eq:inference_model}
    -\frac{d}{dt}p_i(t) = \nabla_{q_i} \phi(q_i) := \frac{1}{n}\sum_{j=1}^{n} \left(q_i(t) - q_j(t)\right) \; \psi\left(\|q_i(t) - q_j(t)\|\right)
\end{equation}
$$

This is true for analytical potentials that have a pairwise dependence -- including the Lennard Jones potential, the Morse Potential, the Stirling Weber potential and variations thereof. Assuming reference data (positions, velocity and forces) can be generated for these $$n$$ particles over $$m$$ timesteps and $$l$$ different initial conditions, the following minimization problem is valid.

$$
\begin{equation}
    \label{eq:minimization_problem}
    \arg \min_{\psi \in \mathcal{H}} \left[ \frac{1}{l}\sum_{k=1}^{l} \frac{1}{m}\sum_{j=1}^{m} \frac{1}{n}\sum_{i=1}^{n} \left\|\frac{d}{dt}p_i(t) + \frac{1}{n}\sum_{p=1}^{n} \left(q_i(t) - q_p(t)\right) \; \psi\left(\|q_i(t) - q_p(t)\|\right)\right\|_2\right]
\end{equation}
$$

--- 

Introducing $$\frac{d}{dt}p_i(t):=f_i$$ and $$\sum_{p=1}^{n} \left(q_i(t) - q_p(t)\right) \; \psi\left(\|q_i(t) - q_p(t)\|\right):=R_i \psi(r_i)$$.

$$
\begin{align}
    \label{eq:lstsq}
    \|f_i + R_i \psi(r_i) \|_2 = \left(f_i + R_i \psi(r_i)\right)^{T} \left(f_i + R_i \psi(r_i)\right) =  \left(f_i^T + \psi(r_i)^T R_i^T\right)\left(f_i + R_i \psi(r_i)\right) \\ 
    = f_i^T f_i + f_i^T R_i \psi(r_i) + \psi(r_i)^T R_i^T f_i + \psi(r_i)^T R_i^T R_i \psi(r_i) 
\end{align}
$$

Then

$$
\begin{align}
    \label{eq:minimizer}
    \frac{d}{d\psi} \|f_i + R_i \psi(r_i) \|_2 = f_i^T R_i + R_i^T f_i + R_i^T R_i \psi(r_i) + \psi(r_i)^T R_i^T R_i 
\end{align}
$$

---

In the complete context of (\ref{eq:minimization_problem}),

$$
\begin{equation}
\frac{1}{l} \frac{1}{m} \frac{1}{n}\sum_{k=1}^{l}\sum_{j=1}^{m}\sum_{i=1}^{n} R_{kji}^T R_{kji} \psi(r_{kji}) + R_{kji}^T f_{kji} \; != 0 
\end{equation}
$$


If $$\psi(r) := \eta^T(r) k$$, then
$$
\begin{equation}
\frac{1}{l} \frac{1}{m} \frac{1}{n}\sum_{k=1}^{l}\sum_{j=1}^{m}\sum_{i=1}^{n}R_{kji}^T R_{kji} \eta(r_{kji})^T k =  -\frac{1}{l}\frac{1}{m}\frac{1}{n} \sum_{k=1}^{l} \sum_{j=1}^{m} \sum_{i=1}^{n} R_{kji}^T f_{kji}
\end{equation}
$$

Equivalently,
$$
\begin{equation}
\label{eq:additive}
\frac{1}{l} \frac{1}{m} \frac{1}{n}\sum_{k=1}^{l}\sum_{j=1}^{m}\sum_{i=1}^{n}A_{kji} k =  -\frac{1}{l}\frac{1}{m}\frac{1}{n} \sum_{k=1}^{l} \sum_{j=1}^{m} \sum_{i=1}^{n} B_{kji}
\end{equation}
$$
$$
\begin{equation}
\mathcal{A}k=\mathcal{B}
\end{equation}
$$

The additive nature of (\ref{eq:additive}) ensures a memory efficient inference procedure for $$k$$ and inturn $$\psi$$.