---
layout: post
title: Constrained MD mean force estimator
date: 2022-10-01 0:00:00
description: This post describe the formulation of the constrained MD mean force estimator.
tags: Physics
citation: true
---

This theme supports rendering beautiful math in inline and display modes using [MathJax 3](https://www.mathjax.org/) engine.

# Constrained MD formulation

The Lagrangian of the system $$\mathcal{L}$$ is extended as:

$$
\begin{equation}
\mathcal{L}^*(\mathbf{q}, \dot{\mathbf{q}})=\mathcal{L}(\mathbf{q}, \dot{\mathbf{q}})+\sum_{i=1}^r \lambda_i \sigma_i(q)
\end{equation}
$$

where the summation is over $$r$$ geometric constraints, $$\mathcal{L}^*$$ is the Lagrangian for the extended system, and $$\lambda_i$$ is a Lagrange multiplier associated with a geometric constraint $$\sigma_i$$:

$$
\begin{equation}
\sigma_i(q)=\xi_i(q)-\xi_i
\end{equation}
$$

with $$\xi_i(q)$$ being a geometric parameter and $$\xi_i$$ is the value of $$\xi_i(q)$$ fixed during the simulation.

In the SHAKE algorithm, the Lagrange multipliers $$\lambda_i$$ are determined in the iterative procedure:

1. Perform a standard MD step (leap-frog algorithm):
   $$
   \begin{equation}
   \begin{aligned}
   v_i^{t+\Delta t / 2} & =v_i^{t-\Delta t / 2}+\frac{a_i^t}{m_i} \Delta t \\
   q_i^{t+\Delta t} & =q_i^t+v_i^{t+\Delta t / 2} \Delta t
   \end{aligned}
   \end{equation}
   $$
2. Use the new positions $$q^{t+\Delta t}$$ to compute Lagrange multipliers for all constraints:
   $$
   \begin{equation}
   \lambda_k=\frac{1}{\Delta t^2} \frac{\sigma_k\left(q^{t+\Delta t}\right)}{\sum_{i=1}^N m_i^{-1} \nabla_i \sigma_k\left(q^t\right) \nabla_i \sigma_k\left(q^{t+\Delta t}\right)}
   \end{equation}
   $$
3. Update the velocities and positions by adding a contribution due to restoring forces (proportional to $$\lambda_k$$):
   $$
   \begin{equation}
   \begin{aligned}
   & v_i^{t+\Delta t / 2}=v_i^{t-\Delta t / 2}+\left(a_i^t-\sum_k \frac{\lambda_k}{m_i} \nabla_i \sigma_k\left(q^t\right)\right) \Delta t \\
   & q_i^{t+\Delta t}=q_i^t+v_i^{t+\Delta t / 2} \Delta t
   \end{aligned}
   \end{equation}
   $$
4. repeat steps 2-4 until either $$|\sigma_i(q)|$$ are smaller than a predefined tolerance, or the number of iterations exceeds a predefined threshold.
