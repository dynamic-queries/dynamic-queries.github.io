---
layout: page
title: para-NN 
description: Learning parallel in time integrators with neural networks.
img: 
importance: 2
category: 2023
---

--- 

## Parallel Time Integration
Consider a system of ordinary differential equations, usually arising from the discretization of a partial differential equation in Eucledian space.

$$
\begin{equation}
    \label{eq:ODE}
    \frac{d}{dt}y(t) = f(y(t),t)
\end{equation}
$$

$$
\begin{equation}
    \label{eq:IC}
    y(0) = y_0 \in \mathbb{R}^d
\end{equation}
$$

Due to the causal nature of the solution to (\ref{eq:ODE}-\ref{eq:IC}), time integration of this problem has been carried out largely sequentially. Parallel in time integration method undertake an alternative approach. 

Here the monolithic equation in (\ref{eq:ODE}-\ref{eq:IC}) is partioned into $$N$$ different sub-problems in the time domain, that is $$\forall \: i \in [0,T-1]$$,

$$
\begin{equation}
    \label{eq:Parallel}
    \frac{d}{dt}y_i(t) = f(y_i(t),t)
\end{equation}
$$

$$
\begin{equation}
    \label{eq:IC Parallel}
    y_{i}(t_i) = Y_i \in \mathbb{R}^d
\end{equation}
$$

$$
\begin{equation}
    \label{eq:IC Parallel Initial}
    Y_0 = y_{i}(0)
\end{equation}
$$

subject to constraints governing the consistent of the $$y_i$$ at the interfaces.

$$
\begin{align}
    \label{eq:constraints}
    Y_0 = y(0) \\ 
    Y_{j} = y_{j-1}(t_{j},Y_{j-1})
\end{align}
$$
where $$j \in [1,T-1]$$. 

If $$Y = [Y_0, Y_1, ..., Y_{T-1}]$$, then the constraints form a non-linear system of equations.

$$
\begin{equation}
    \label{eq:non-linear}
    F(Y) = 0
\end{equation}
$$

whose solution is promptly obtained from Newton's iteration.

$$
\begin{equation}
    \label{eq:Newton}
    Y^{k+1} = Y^{k} - \nabla_Y F(Y^{k})^{-1} F(Y^k)
\end{equation}
$$

The Jacobian for (\ref{eq:non-linear}) has a sparse, diagonal structure
$$
\begin{equation}
    \label{eq:jac}
    J := \nabla_Y F = \begin{bmatrix}
    I& & & & &\\
    \frac{\partial y_1}{\partial Y_0}&I&&&& \\
    &\frac{\partial y_2}{\partial Y_1}&I&&& \\ 
    &&\ddots &\ddots&\\
    &&&\frac{\partial y_{T-1}}{\partial Y_{T-2}}&I \\
    \end{bmatrix}
\end{equation}
$$
yielding a closed form solution to (\ref{eq:Newton}).

$$
\begin{equation}
    \label{eq:closed-form-1}
    Y_0^{k+1} = y(0)
\end{equation}
$$
$$
\begin{equation}
    \label{eq:closed-form-2}
    Y_n^{k+1} = y_{n-1}(t_n, Y_{n-1}^k) + \frac{\partial y_{n-1}}{\partial Y_{n-1}}(t_n, Y_{n-1}^k) (Y_{n-1}^{k+1}-Y_{n-1}^{k})
\end{equation}
$$

This equation is still implicit. 


## Contributions 
Usually, $$y_j$$ in (\ref{eq:closed-form-2}) are evaluated from one-step integrators such as **RK4, Tsit5** and the like. Here, we parametrize $$y_j$$ using a neural network with an explicit time integrator structure, namely $$y_{NN}$$. Subsequently, the derivative $$\frac{\partial y_{j}}{\partial Y_{j}}$$ is computed with automatic differentiation, alleviating the need for explicity discretizing $$\frac{\partial y_{j}}{\partial Y_{j}}$$. It is expected that, the learned time-integrator, is customized for parallel execution and for the predefined problem, whose solutions were used for training.