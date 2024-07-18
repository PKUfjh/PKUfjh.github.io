---
layout: post
title: String method
description: This post describes string method.
date: 2024-07-18 0:00:00
tags: Physics
---

# Minimum free energy path
Consider a system in the NVT ensemble, the equilibirum distribution is given by:
$$
\begin{equation}
\rho(x)=Z^{-1} e^{-\beta V(x)}
\end{equation}
$$
Suppose that one introduces N collective variables,
$$
\begin{equation}
\theta(x)=\left(\theta_1(x), \ldots, \theta_N(x)\right)
\end{equation}
$$
then the free energy associated with the collective variables is:
$$
\begin{equation}
\begin{aligned}
F(z)= -k_B T \ln \left(Z^{-1} \int_{\mathbb{R}^n} e^{-\beta V(x)} \times \delta\left(z_1-\theta_1(x)\right) \cdots \delta\left(z_N-\theta_N(x)\right) d x\right) .
\end{aligned}
\end{equation}
$$
Let us represent the MEP by the curve $$x(\alpha)$$, where $$\alpha \in [0,1]$$ is the parameter used to parametrize the curve. Since
by definition the force $$ - \nabla V$$ must be everywhere tangent to the MEP, we must have
$$
\begin{equation}
\label{eq:1}
\frac{d x_k(\alpha)}{d \alpha} \text { parallel to } \frac{\partial V(x(\alpha))}{\partial x_k} \text {. }
\end{equation}
$$
If the MEP connects the two minima of $$V(x)$$ located at $$x_a$$
and $$x_b$$, Eq. \ref{eq:1} must be supplemented by the boundary conditions $$x(0)=x_a$$ and $$x(1)=x_b$$.

Let us now consider things in the space of $$z = \theta(x)$$, Eq. \ref{eq:1} implies that:
$$
\begin{equation}
\label{eq:2}
\begin{aligned}
\frac{d z_i(\alpha)}{d \alpha}= & \sum_{k=1}^n \frac{\partial \theta_i(x(\alpha))}{\partial x_k} \frac{d x_k(\alpha)}{d \alpha} \\
& \text { parallel to } \sum_{k=1}^n \frac{\partial \theta_i(x(\alpha))}{\partial x_k} \frac{\partial V(x(\alpha))}{\partial x_k} \\
= & \sum_{j, k=1}^n \frac{\partial \theta_i(x(\alpha))}{\partial x_k} \frac{\partial \theta_j(x(\alpha))}{\partial x_k} \frac{\partial U(z(\alpha))}{\partial z_j} .
\end{aligned}
\end{equation}
$$
It is shown in later section that it amounts to replacing $$U(z)$$ by $$F(z)$$ and the tensor $$\sum_k (\partial \theta_i / \partial x_k) \times (\partial \theta_j / \partial x_k)$$ by its average $$M_{ij}$$ defined as:
$$
\begin{equation}
\begin{aligned}
M_{i j}(z)= & Z^{-1} e^{\beta F(z)} \int_{\mathbb{R}^n} \sum_{k=1}^n \frac{\partial \theta_i(x)}{\partial x_k} \frac{\partial \theta_j(x)}{\partial x_k} e^{-\beta V(x)} \\
& \times \delta\left(z_1-\theta_1(x)\right) \cdots \delta\left(z_N-\theta_N(x)\right) d x .
\end{aligned}
\end{equation}
$$
Making these substutions into Eq. \ref{eq:2}, we arrive at:
$$
\begin{equation}
\frac{d z_i(\alpha)}{d \alpha} \text { parallel to } \sum_{j=1}^N M_{i j}(z(\alpha)) \frac{\partial F(z(\alpha))}{\partial z_j} \text {. }
\end{equation}
$$
which also means:
$$
\begin{equation}
0=\sum_{j, k=1}^N P_{i j}(\alpha) M_{j k}(z(\alpha)) \frac{\partial F(z(\alpha))}{\partial z_k}
\end{equation}
$$
where $$P_{i j}$$ is the projector on the plane perpendicular to the path at $$z(\alpha)$$,
$$
\begin{equation}
\label{eq:3}
P_{i j}(\alpha)=\delta_{i j}-\hat{t}_i(\alpha) \hat{t}_j(\alpha), \quad \hat{t}_i(\alpha)=\frac{\partial z_i / \partial \alpha}{\left|\partial z_i / \partial \alpha\right|}
\end{equation}
$$
How to
find this solution in practice is explained in the next section.

# String method and mean force calculation
To solve Eq. \ref{eq:3}, The simplest way to evolve $$z(\alpha, t)$$ so that it converges to a MFEP is to use,
$$
\begin{equation}
\frac{\partial z_i(\alpha, t)}{\partial t}=-\sum_{j, k=1}^N P_{i j}(\alpha, t) M_{j k}(z(\alpha, t)) \frac{\partial F(z(\alpha, t))}{\partial z_k},
\end{equation}
$$
