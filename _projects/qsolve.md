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

For the rest of this discussion, I will discuss common ways to obtain orthogonal duals for a given matrix $$M$$, discuss their computational complexity and illustrate its place in the landscape of linear solvers.

## Orthogonalization

There are countless orthogonalization schemes that are available in literature. Some of the commonly used ones include: