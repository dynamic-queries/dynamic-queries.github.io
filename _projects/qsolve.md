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

## Prelude 
Consider a linear system $$Ax=b$$ where $$x,b \in \mathbb{R}^d$$ and $$A \in \mathbb{R}^{d \times d}$$. 

Geometrically, this equation can be viewed as determining the projections of a given vector $$b$$ in some vector space $$\mathcal{S}$$ onto a different bases set $$A$$ that also spans $$\mathcal{S}$$. So in a way, the task of solving a linear system, is the task of orthogonalizing $$A$$. 

If $$Q$$ is orthogonal to $$A$$ ,that is, $$QA = \mathcal{I}$$, then
$$
\begin{align}
    Ax = b \\ 
    QAx = Qb \\ 
    x = Qb
\end{align} 
$$

## Orthogonalization

There are countless orthogonalization schemes that are available in literature. Some of the commonly used ones include:

<div class="alert alert-block alert-success">
   1. Gram Schmidt projection <br>
   2. Householder reflection <br>
</div>

### Gram Schmidt projection

Let $$A = \left[ a_1\:a_2\:a_3 \:...\: a_n \right]$$ be composed of $$n$$ different directions enumerated by $$\{a_i:i \in (1,n)\}$$. The Gram-Schmidt procedure recursively generates orthonogonal vectors $$v_i$$ using the method of projections.

$$
\begin{equation}
    v_1 = a_1
\end{equation}
$$


$$
\begin{equation}
    v_2 = a_2 - \frac{\langle a_2 | v_1\rangle}{\langle v_1 | v_1\rangle} 
\end{equation}
$$

$$
\begin{equation}
    v_n = a_n - \sum_{j=1}^{n-1} \frac{\langle a_n | v_j\rangle}{\langle v_{j} | v_{j}\rangle} 
\end{equation}
$$

It is not hard to demonstrate that the Grad-Schmidt procedure scales as $$\mathcal{O}(n^3)$$. Furthermore, parallelizing the method is not possible due to its formulation. 

## Householder reflection



Owing to these inherent difficulties, solving linear systems with orthogonalization methods is suboptimal in comparison with other Kyrlov based iterative methods. Nonetheless it serves as a nice method which could be geometrically interpreted.