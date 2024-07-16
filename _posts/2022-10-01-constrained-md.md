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
4. repeat steps 2-4 until either $$\mid \sigma_i(q) \mid$$ are smaller than a predefined tolerance, or the number of iterations exceeds a predefined threshold.

# Free energy and mean force definition

The definition for free energy w.r.t some predefined collective varibales (CVs) is:

$$
\begin{equation}
A(s)=-k_b \operatorname{Tln} p(s)=-\frac{1}{\beta} \ln \int \frac{1}{Z} e^{-\beta U(r)} \Pi_i \delta\left(s_i(r)-s_i\right) d r
\end{equation}
$$

Accordingly, the mean force is the negative gradient of the free energy:

$$
\begin{equation}
- \nabla_{s_i} A(s)= - \frac{1}{\beta Z} \int e^{-\beta U(r)} \nabla_{s_i} \Pi_j \delta\left(s_j(r)-s_j\right) d r
\end{equation}
$$

# Mean force estimator for constraiend MD

We show that if there exists $$b_i(r)$$, such that:

$$
\begin{equation}
\label{eq:condition}
\nabla_r s_i(r) \cdot b_j(r)=\delta_{i j}
\end{equation}
$$

then the mean force estimator expression is:

$$
\begin{equation}
-\nabla_{s_i} A(s)=\frac{1}{Z} \int e^{-\beta U(r)}\left(-\nabla U(r) \cdot b_i(r)+\frac{1}{\beta} \nabla \cdot b_i(r)\right) \Pi_j \delta\left(s_j(r)-s_j\right) d r
\end{equation}
$$

The proof is in the following, from \ref{eq:condition} we can get:
$$
\begin{equation}
b_j(r) \cdot \nabla_r \delta\left(s_i(r)-s_i\right)=b_j(r) \cdot \nabla_r\left(s_i(r)\right) \cdot \nabla_{s_i} \delta\left(s_i(r)-s_i\right)=\delta_{i j} \cdot \nabla_{s i} \delta\left(s_i(r)-s_i\right)
\end{equation}
$$
