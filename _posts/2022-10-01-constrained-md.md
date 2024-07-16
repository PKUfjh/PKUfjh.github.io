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

The proof is in the following, from Eq. (\ref{eq:condition}) we can get:

$$
\begin{equation}
b_j(r) \cdot \nabla_r \delta\left(s_i(r)-s_i\right)=b_j(r) \cdot \nabla_r\left(s_i(r)\right) \cdot \nabla_{s_i} \delta\left(s_i(r)-s_i\right)=\delta_{i j} \cdot \nabla_{s i} \delta\left(s_i(r)-s_i\right)
\end{equation}
$$

Then:

$$
\begin{equation}
b_i(r) \cdot \nabla_r \Pi_j \delta\left(s_j(r)-s_j\right)=\nabla_{s_i} \delta\left(s_i(r)-s_i\right) \Pi_{j \neq I} \delta\left(s_j(r)-s_j\right)
\end{equation}
$$

So:

$$
\begin{equation}
\begin{aligned}
\nabla_{s_i} A(s)& =\frac{1}{\beta Z} \int e^{-\beta U(r)} \nabla_{s_i} \Pi_j \delta\left(s_j(r)-s_j\right) d r=\frac{1}{\beta Z} \int e^{-\beta U(r)} b_i(r) \nabla_r \Pi_j \delta\left(s_j(r)- s_j\right) d r \\
& =\frac{1}{\beta Z} \int e^{-\beta U(r)}\left(\beta \nabla U(r) \cdot b_i(r)-\nabla \cdot b_i(r)\right) \Pi_j \delta\left(s_j(r)-s_j\right) d r \\
& =<\nabla U(r) \cdot b_i(r)-\frac{1}{\beta} \nabla \cdot b_i(r)>_{\text {cond }}
\end{aligned}
\end{equation}
$$

So:

$$
\begin{equation}
\begin{aligned}
& f_i=-\nabla_{s_i} A(s)=<-\nabla U(r) \cdot b_i(r)+\frac{1}{\beta} \nabla \cdot b_i(r)>_{\text {cond }}=<f(r) \cdot b_i(r)+\frac{1}{\beta} \nabla b_i(r)>_{\text {cond }}
\end{aligned}
\end{equation}
$$

This mean force average, like any other observables, however, is not the same with the
"average" taken from a "constrained md" trajectory. A one sentence explanation is: The dynamics of constrianed MD is non-Hamiltonian, namely the equation of motion of constrained MD does not preserve volume element due to the non-zero "compressibility", so we have to take a factor into the "conditional average", to get the "constrained MD average".

# Constrained ensemble

To see this more clearly, we have to go back to the definition of constrained MD. The equation of motion for constrained MD is,

$$
\begin{equation}
\begin{aligned}
\dot{x_j} & =M^{-1} p_j \\
\dot{p_j} & =-\frac{\partial V}{\partial x_j}+\frac{\partial \sigma}{\partial x_j} \lambda(x, p)
\end{aligned}
\end{equation}
$$

The mass matrix is diagonal in general, so I do not write lower index. From theoretical mechanics, it is easy to see that the evolution of the jacabian in phase space is,

$$
\begin{equation}
     \frac{d J}{d t}=J\left(\partial_j \dot{\xi}^j\right)
\end{equation}
$$

So if we define "compressibility" to be:

$$
\begin{equation}
\kappa=\partial_j \dot{\xi}^j
\end{equation}
$$

Then the evolution of the jacobian is:

$$
\begin{equation}
J(\xi(t) ; \xi(0))=\exp \left(\int_0^t d s \kappa(\xi(s))\right)=\exp (W(\xi(t))-W(\xi(0))
\end{equation}
$$

If we define a "metric factor":

$$
\begin{equation}
\sqrt{g(t)} \equiv e^{-W(\xi(t))}
\end{equation}
$$

Then we can still get a constant "volume element":

$$
\begin{equation}
\sqrt{g(t)} d^{2 n} \xi(t)=\sqrt{g(0)} d^{2 n} \xi(0)
\end{equation}
$$

This is equivalent to bring the metric factor into the "conditonal average", namely,

$$
\begin{equation}
\langle\mathcal{O}(\mathbf{r})\rangle_s^{\text {constr }}=\frac{\int \mathrm{d} \mathbf{r} \mathrm{e}^{-\beta U(\mathbf{r})} z^{1 / 2}(\mathbf{r}) \mathcal{O}(\mathbf{r}) \delta\left(f_1(\mathbf{r})-s\right)}{\left\langle z^{1 / 2}(\mathbf{r}) \delta\left(f_1(\mathbf{r})-s\right)\right\rangle}
\end{equation}
$$

Rememble the "conditional average" is,

$$
\begin{equation}
\langle\mathcal{O}(\mathbf{r})\rangle_s^{\text {cond }}=\frac{\int \mathrm{dre}^{-\beta U(\mathbf{r})} \mathcal{O}(\mathbf{r}) \delta\left(f_1(\mathbf{r})-s\right)}{\left\langle\delta\left(f_1(\mathbf{r})-s\right)\right\rangle}
\end{equation}
$$

So if we would like to use the trajectory from constrained MD to estimate mean force in the
conditional average expression, we have to bring a metrix factor in, namely,

$$
\begin{equation}
\langle\mathcal{O}(\mathbf{r})\rangle_s^{\text {cond }}=\frac{\left\langle z^{-1 / 2}(\mathbf{r}) \mathcal{O}(\mathbf{r})\right\rangle_s^{\text {constr }}}{\left\langle z^{-1 / 2}(\mathbf{r})\right\rangle_s^{\text {constr }}}
\end{equation}
$$

# Phase factor

Then it is a matter of calculating the phase factor (metric factor ) in constrained MD.
From the analysis of the previous section, to calculate the metric factor we need to calculate,

$$
\begin{equation}
\partial_j \dot{\xi}^j \text {, where } \xi^j=\left(q^j, p^j\right) \text {. }
\end{equation}
$$

From the equation of motion, this amounts to calculate $$\partial_{p_i} \frac{\partial \sigma}{\partial x_i} \lambda (x,p)$$
I first give the answer and will write the derivation later. The phase factor equals:

$$
\begin{equation}
z^{1 / 2}=\left(\operatorname{det}\left[\frac{\partial \sigma_m}{\partial x_j} M^{-1} \frac{\partial \sigma_n}{\partial x_j}\right]\right)^{1 / 2}
\end{equation}
$$

Where $$\sigma_m$$ is the mth constraints exerted on the system.
