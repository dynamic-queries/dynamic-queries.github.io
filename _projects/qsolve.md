---
layout: page
title: QSolve
description: Quantum Algorithms for solving linear systems.
img: 
importance: 5
category: 2020
---

Solving linear systems is at the heart of all scientific computing applications. It is based on this notion, in mathematics, of converting a new problem into a problem that you already know how to solve. Incidentally, the field of machine learning (generative models with diffusion), statistics and other natural sciences also have vested interests in improving the state of the art of linear solvers.

In this work, I investigate the applicability of solving partial differential equations using linear solvers designed for quantum computers.

#### Problem setup 
Consider a linear system $$Ax=b$$ where $$x,b \in \mathbb{R}^d$$ and $$A \in \mathbb{R}^{d \times d}$$. 

Geometrically, this equation can be viewed as determining the projections of a given vector $$b$$ in some vector space $$\mathcal{S}$$ onto a different bases set $$A$$ that also spans $$\mathcal{S}$$. So in a way, the task of solving a linear system, is the task of orthogonalizing $$A$$. 

#### Orthogonalization -- the trivial solution
If $$Q$$ is orthogonal to $$A$$ ,that is, $$QA = \mathcal{I}$$, then
$$
\begin{align}
    Ax = b \\ 
    QAx = Qb \\ 
    x = Qb
\end{align} 
$$

There are countless orthogonalization schemes that are available in literature. Some of the commonly used ones include:

<div class="alert alert-block alert-success">
   1. Gram Schmidt projection <br>
   2. Householder reflection <br>
</div>


Despite the ideological simplicity, solving linear systems with orthogonalization methods is suboptimal in comparison with other Kyrlov based iterative methods. Nonetheless it serves as a nice method which could be geometrically interpreted. For a more detailed exposition, see [Kelly](https://people.math.sc.edu/Burkardt/m_src/kelley/kelley.html). 

#### HHL algorithms -- the quantum solution
Alternatively, there exist a family of algorithms called the HHL algorithms that provide exponential speedups over classical algorithms. Generally, HHL algorithms have a computational complexity of $$\mathcal{O}(log(d))$$ compared to the $$\mathcal{O}(d^2)$$ of conjugate gradient methods.

HHL algorithms leverage the following property of the spectrum of $$A$$.

Let 

$$
\begin{equation}
    \label{eq:Eig}
    A p_i = \lambda_i p_i
\end{equation}
$$ 

be the eigenproblem with eigenvalues -- $$\{\lambda_i\}_{i=1}^{d}$$ and eigenvectors -- $$\{p_i\}_{i=1}^{d}$$. Then the inverse operator $$A^{-1}$$ satisfies

$$
\begin{equation}
    A^{-1} p_i = \frac{1}{\lambda_i} p_i
\end{equation}
$$

Ideally, if one solves the eigenproblem in (\ref{eq:Eig}), then inverting a matrix is as trivial as computing the reciprocal of its eigenvalues. For the rest of this section, I will discuss the bare-bone algorithms that will be used for efficiently performing this inversion.

##### 1. Quantum Fourier Transform
This algorithm is used to compute the Fourier transform of function on a quantum computer. Let $$q \in \mathbb{R}^m$$ be a vector (discrete version of a function). Its discrete Fourier transform is given by

$$
\begin{equation}
    \mathcal{F}(q_k) = \sum_{j=0}^{m-1} exp(i\:2\pi\frac{jk}{m}) \: q_j
\end{equation}
$$

If $$\hat{q_j}$$ are the individual bases in the real space, then let the fourier bases be denoted by $$\hat{p_k}$$. Such that $$\hat{p} = \sum_k \alpha_{p_k} \hat{p_k}$$ and $$\hat{q} = \sum_j \alpha_{q_j} \hat{q_j}$$.

Analogously, the quantum Fourier transform can be written in terms of its orthogonal bases as:

$$
\begin{equation}
    \hat{p} := \mathcal{F}(\hat{q}) = \sum_j \alpha_{q_j} \mathcal{F} \left[ \hat{q_j} \right]
\end{equation}
$$

$$
\begin{equation}
    \mathcal{F} \left[ \hat{q_j} \right] = \sum_k exp(i\:2\pi\frac{q_j k}{m}) \: \hat{k}
\end{equation}
$$

where $$q_j,k$$ are components of $$\hat{q_j},\hat{k}$$ respectively.

If $$m = 2^n$$

$$
\begin{equation}
    \mathcal{F} \left[ q_j \right] = \sum_k exp(i\:2\pi\frac{q_j k}{2^n}) \: \hat{k}
\end{equation}
$$

Let $$k = \sum_{l=0}^{n-1} 2^l \: k_l$$ and $$\frac{k}{2^n} = \sum_{j=1}^{n} 2^{-l} \hat{k_l}$$, then 

$$
\begin{equation}
    \sum_k exp(i\:2\pi\frac{q_j k}{2^n}) \: k = \sum_{k_1 \in \{0,1\}} \sum_{k_2 \in \{0,1\}}...\sum_{k_n \in \{0,1\}} exp(i\:2\pi q_j \sum_{l=1}^{n} 2^{-l} k_l) \bigotimes_{l=1}^{n} \hat{k_l} 
\end{equation}
$$

$$
\begin{equation}
    \sum_k exp(i\:2\pi\frac{q_j k}{2^n}) \: k = \sum_{k_1 \in \{0,1\}} \sum_{k_2 \in \{0,1\}}...\sum_{k_n \in \{0,1\}} \bigotimes_{l=1}^{n} exp(i\:2\pi q_j  2^{-l} k_l) \bigotimes_{l=1}^{n} \hat{k_l}
\end{equation}
$$

$$
\begin{equation}
    \sum_k exp(i\:2\pi\frac{q_j k}{2^n}) \: k = \sum_{k_1 \in \{0,1\}} \sum_{k_2 \in \{0,1\}}...\sum_{k_n \in \{0,1\}} \bigotimes_{l=1}^{n} exp(i\:2\pi q_j  2^{-l} k_l) \hat{k_l} 
\end{equation}
$$

$$
\begin{equation}
    \sum_k exp(i\:2\pi\frac{q_j k}{2^n}) \: k = \bigotimes_{l=1}^{n} \sum_{k_l \in \{0,1\}} exp(i\:2\pi q_j  2^{-l} k_l) \hat{k_l} 
\end{equation}
$$

$$
\begin{equation}
    \sum_k exp(i\:2\pi\frac{q_j k}{2^n}) \: k = \bigotimes_{l=1}^{n} \left[ \hat{0} + exp(i\:2\pi q_j  2^{-l}) \hat{1} \right]
\end{equation}
$$

This transformation allows one to efficiently write down QFT, efficiently as linear algebraic operations (alternatively as a quantum circuit). I will discuss this computation in the rest of the section.

##### 2. Quantum Phase Estimation

##### 3. Quantum Scalar Inversion

#### Solving PDEs with HHL algorithms

##### Diffusion equation


##### Wave equation