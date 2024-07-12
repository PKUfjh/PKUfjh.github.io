---
layout: post
title: All you need to know about free energy (Part I)
date: 2023-11-07 0:00:00
tags: Physics
---

# Fundamental Theory of Free Energy

## What is Free Energy

Thermodynamic perspective (at constant temperature and volume):

$$F = U - TS$$

The (Helmholtz) free energy $$F$$ of a system consists of two parts: the internal energy $$U$$ and the entropy $$S$$ (degree of disorder) of the system. According to the second law of thermodynamics, an isolated system will always evolve towards a state of increased entropy. In a constant temperature and volume scenario, the second law of thermodynamics states that a system will evolve towards a state where the free energy $$F$$ decreases. From the above formula, we can see that this trend is determined by two state quantities: 1. The system tends to evolve towards a state with lower internal energy $$U$$. 2. The system tends to evolve towards a state with higher entropy $$S$$. Due to the presence of the temperature $$T$$ coefficient in front of the entropy term, we can immediately see that at higher temperatures, the system's state change is more influenced by entropy, while at lower temperatures, it is more influenced by internal energy.

Statistical mechanics perspective:

$$
F = - \frac{1}{\beta} \ln Z, Z =\int \mathrm{d}^N \mathbf{r} \mathrm{e}^{-\beta E \left(\mathbf{r}_1, \ldots, \mathbf{r}_N\right)}
$$

where $$\beta = \frac{1}{k_B T}$$, $$Z$$ is the partition function of the system. From the statistical mechanics definition of free energy $$F$$, it is evident that the free energy of the system is entirely determined by the magnitude and distribution of energy $$E$$ in the coordinate space. To reduce the free energy, the system will tend to adopt states with lower energy $$E$$. Moreover, due to the presence of the integral sign, the system will tend to form states with a higher number of microstates at the same energy level, which corresponds to entropy in thermodynamics.

## Calculation of Free Energy

From the aforementioned definitions, a direct observation is that the free energy $$F$$ of a system depends on the definition of internal energy $$U$$. Since the definition of energy is relative, the absolute value of the system's free energy $$F$$ is also meaningless. Typically, we are almost always concerned with the change in free energy between two states of the system, namely:

$$
\Delta F = F_B - F_A
$$

According to the statistical mechanics definition of free energy, the change in free energy can be written as:

$$
\Delta F = - k_B T \ln Z_B + k_B T \ln Z_A = - k_B T \ln \frac{Z_B}{Z_A}
$$

where $$Z_A$$ and $$Z_B$$ are the partition functions of the system in states A and B, respectively:

$$
\begin{aligned}
Z_{\mathcal{A}} & =\int \mathrm{d}^N \mathbf{r} \mathrm{e}^{-\beta U_{\mathcal{A}}\left(\mathbf{r}_1, \ldots, \mathbf{r}_N\right)} \\
Z_{\mathcal{B}} & =\int \mathrm{d}^N \mathbf{r} \mathrm{e}^{-\beta U_{\mathcal{B}}\left(\mathbf{r}_1, \ldots, \mathbf{r}_N\right)}
\end{aligned}
$$

This concludes the introduction to the definition of free energy calculation. The following sections will delve into how to compute it, specifically focusing on how to use molecular dynamics simulations to calculate free energy, as this is a tutorial on molecular dynamics.

If we know the form of the energy function $$U(r)$$ for the system in states A and B, then calculating the partition function is straightforward. However, in general, we do not know this, which is one reason why we use molecular dynamics as a tool. For molecular dynamics, we usually cannot obtain the partition function for a system in a particular state. Instead, we can obtain the ensemble distribution (typically at constant temperature and volume) and ensemble average (time average) of a physical quantity in that state. Therefore, the core idea of using molecular dynamics to obtain the change in the system's state free energy is to convert the free energy change into an ensemble average of some physical quantity, which is the starting point for most free energy calculation methods.

Next, we will look at two main approaches to free energy calculation: equilibrium methods and non-equilibrium methods.

## Equilibrium Free Energy Calculation Methods

### Free Energy Perturbation (FEP)

To write the change in the system's free energy as an ensemble average of some physical quantity, we note that the partition function part of the free energy change can be written as:

$$
\begin{aligned}
\frac{Z_{\mathcal{B}}}{Z_{\mathcal{A}}} & =\frac{1}{Z_{\mathcal{A}}} \int \mathrm{d}^N \mathbf{r} \mathrm{e}^{-\beta U_{\mathcal{A}}\left(\mathbf{r}_1, \ldots, \mathbf{r}_N\right)} \mathrm{e}^{-\beta\left(U_{\mathcal{B}}\left(\mathbf{r}_1, \ldots, \mathbf{r}_N\right)-U_{\mathcal{A}}\left(\mathbf{r}_1, \ldots, \mathbf{r}_N\right)\right)} \\
& =\left\langle\mathrm{e}^{-\beta\left(U_{\mathcal{B}}\left(\mathbf{r}_1, \ldots, \mathbf{r}_N\right)-U_{\mathcal{A}}\left(\mathbf{r}_1, \ldots, \mathbf{r}_N\right)\right)}\right\rangle_{\mathcal{A}}
\end{aligned}
$$

Therefore, the change in free energy of the system becomes:

$$
\Delta F_{\mathcal{A B}}=-k T \ln \left\langle\mathrm{e}^{-\beta\left(U_{\mathcal{B}}-U_{\mathcal{A}}\right)}\right\rangle_{\mathcal{A}}
$$

From the perspective of molecular dynamics sampling, this formula can be interpreted as: we sample the potential energy surface of the system in state A, obtaining a series of configurations in the ensemble distribution of the system in state A. For each such configuration, we calculate $$\mathrm{e}^{-\beta\left(U_{\mathcal{B}}-U_{\mathcal{A}}\right)}$$, and the average of these configurations gives the ensemble average of this physical quantity.

Note that the definitions of potential energy in states A and B, $$U_A$$ and $$U_B$$, are different. Consequently, low-energy configurations in state A may not be low-energy configurations in state B. When states A and B differ significantly, the energy calculated in state B for configurations sampled in state A may be high-energy configurations, resulting in small weight factors $$\mathrm{e}^{-\beta\left(U_{\mathcal{B}}-U_{\mathcal{A}}\right)}$$. Due to the limited sampling time in molecular dynamics, we cannot sample configurations with high weight factors, leading to an efficiency problem in our calculations. If states A and B differ significantly, it takes a very long time to obtain a convergent free energy difference estimate.

To solve this sampling efficiency problem, our solution is: since free energy is an equilibrium state property, we can design a path from state A to state B and calculate the free energy difference between adjacent states along this path. Then, the free energy difference between states A and B can be expressed as:

$$
\Delta F_{\mathcal{A B}}= \sum_{\alpha=1}^{M-1}  F_{\alpha+1} - F_{\alpha} = -k T \sum_{\alpha=1}^{M-1} \ln \left\langle\mathrm{e}^{-\beta \Delta U_{\alpha, \alpha+1}}\right\rangle_\alpha
$$

Since the differences between adjacent states on the path are relatively small, their free energy differences converge more quickly. We can sample all the states on the path simultaneously, obtaining a convergent free energy difference more quickly, which is the free energy perturbation method.

### BAR/MBAR

In the derivation of the free energy perturbation theory, we find that a crucial step is the transformation of the ratio of partition functions:

$$
\begin{aligned}
\frac{Z_{\mathcal{B}}}{Z_{\mathcal{A}}} & =\frac{1}{Z_{\mathcal{A}}} \int \mathrm{d}^N \mathbf{r} \mathrm{e}^{-\beta U_{\mathcal{A}}\left(\mathbf{r}_1, \ldots, \mathbf{r}_N\right)} \mathrm{e}^{-\beta\left(U_{\mathcal{B}}\left(\mathbf{r}_1, \ldots, \mathbf{r}_N\right)-U_{\mathcal{A}}\left(\mathbf{r}_1, \ldots, \mathbf{r}_N\right)\right)} \\
& =\left\langle\mathrm{e}^{-\beta\left(U_{\mathcal{B}}\left(\mathbf{r}_1, \ldots, \mathbf{r}_N\right)-U_{\

mathcal{A}}\left(\mathbf{r}_1, \ldots, \mathbf{r}_N\right)\right)}\right\rangle_{\mathcal{A}}
\end{aligned}
$$

This transformation is not unique. For example, we can adopt the following transformation:

$$
\frac{Z_B}{Z_A}=\frac{Z_B}{Z_A} \frac{\int W e^{-U_B-U_A} d \mathbf{q}}{\int W e^{-U_B-U_A} d \mathbf{q}}=\frac{\left\langle W e^{-U_B}\right\rangle_A}{\left\langle W e^{-U_A}\right\rangle_B}
$$

In this transformation, any choice of the function $$W$$ is permissible. Specifically, if we choose $$W = e^{U_A}$$, this transformation reverts to the equation in the aforementioned FEP method.

Now, the question arises: what form of the $$W$$ function achieves the optimal transformation? More precisely, what choice allows us to achieve the best convergence when performing molecular dynamics sampling using this transformation? To address this issue, the Bennett Acceptance Ratio (BAR) method was developed.

BAR, which stands for Bennett Acceptance Ratio, is a method proposed by Bennett in 1976 for calculating free energy. The core of this method lies in selecting the form of the $$W$$ function. For a detailed derivation, see [Bennet 1976](https://dasher.wustl.edu/chem478/reading/jcompphys-22-245-76.pdf). Below, we present the result directly. Suppose we use molecular dynamics simulations to sample the system in state A with a sample size of $$N_A$$ and the system in state B with a sample size of $$N_B$$. The optimal choice of the $$W$$ function to minimize the variance of the partition function ratio in this sampling is:

$$W = \frac{1}{\frac{Z_B}{N_B} e^{-U_A}+\frac{Z_A}{N_A} e^{-U_B}} $$

Note that the $$W$$ function depends on the values of $$Z_B$$ and $$Z_A$$, making the $$W$$ function and the partition functions interdependent. In practice, a self-consistent iterative calculation is required.

Using the BAR method, we can calculate the free energy difference between two states. Although the choice of the $$W$$ function is optimal for the existing samples in both states, the problem of low sampling efficiency remains when states A and B differ significantly. Can we emulate FEP to calculate the free energy difference between multiple states? The Multistate Bennett Acceptance Ratio (MBAR) method is a multistate extension of the BAR method.

In summary, the MBAR method addresses the following problem: given a system with $$K$$ states, we have a series of samples for each of these $$K$$ states, with the number of samples for each state denoted by $$N_i$$. How can we optimally calculate the free energy difference between pairs of states, i.e.,

$$
\Delta F_{i j}=-\beta^{-1} \ln \frac{Z_j}{Z_i}
$$

Just like the BAR method, we have transformations for the ratio of partition functions:

$$
Z_i\left\langle\alpha_{i j} \exp \left(-\beta U_j\right)\right\rangle_i=Z_j\left\langle\alpha_{i j} \exp \left(-\beta U_i\right)\right\rangle_j
$$

The MBAR method concludes with the optimal selection of the coefficients $$\alpha_{i j}$$:

$$
\alpha_{i j}=\frac{N_j{\hat{z_j}}^{-1}}{\sum_{k=1}^K N_k{\hat{z_k}}^{-1} \exp \left(-\beta U_k\right)}
$$

where $$\hat{z_i}$$ is our current estimate of the partition functions based on the samples. This is still a self-consistent iterative calculation formula.

### Hamiltonian Thermodynamic Integration

We have introduced two common free energy calculation methods, and we can see that the MBAR method is an extension of the FEP method. Next, let us consider free energy calculation from a more general perspective.

The two methods introduced earlier, FEP and MBAR, both require sampling multiple states of a system and calculating the potential energy functions. During this process, we need to note that the potential energy functions change for different states of the system. This is crucial, and it is the basis for performing free energy calculations. More precisely, for practical system calculations, the form of the potential energy function is:

$$U = U(\mathbf{r}, \lambda (\mathbf{r}))$$

Alternatively, different states of a system essentially have different Hamiltonians:

$$H = H(\mathbf{r}, \lambda (\mathbf{r}))$$

The free energy difference between different states, or the free energy of different states, is actually a function of the variable $$\lambda$$ in this function. Specifically, if we calculate the binding free energy of a ligand and receptor, in the initial state A, where the ligand and receptor are separated, the Hamiltonian of the entire system does not include the interaction function between the ligand and receptor. In the final state B, where the ligand and receptor are bound, the Hamiltonian of the system includes the interaction function between the ligand and receptor. This interaction is gradually turned on.

In the two methods introduced earlier, FEP and MBAR, the change in the Hamiltonian for different states of the system is implicit, achieved by changing the configuration of the system and implicitly changing the Hamiltonian using the built-in potential energy function of MD. In other words, $$\lambda$$ here is an implicit variable. The free energy obtained for different states does not indicate which specific variable of the system it is a function of; we only know it is a variable related to the overall state of the system. Can we explicitly write out the change in the Hamiltonian for different states of the system? Of course, we can. Since free energy is a state function of the system, we can design a path to let the system evolve from state A to state B.

We define the following series of Hamiltonians:

$$
\mathcal{H}(\mathbf{x}, \lambda)=(1-\lambda) \mathcal{H}_A+\lambda \mathcal{H}_B
$$

Since the system Hamiltonian explicitly contains the variable $$\lambda$$, the system's free energy naturally contains $$\lambda$$:

$$
F(N, V, T, \lambda)=-k_B T \ln Z(N, V, T, \lambda)
$$

From this explicit expression, we can directly write the derivative of free energy with respect to the variable $$\lambda$$:

$$
\begin{aligned}
\frac{\partial F}{\partial \lambda} & =-k_B T \frac{\frac{\partial}{\partial \lambda} Z(N, V, T, \lambda)}{Z(N, V, T, \lambda)} \\
& =-k_B T \frac{\int \exp [-\beta \mathcal{H}(\mathbf{x}, \lambda)] \cdot(-\beta) \frac{\partial \mathcal{H}}{\partial \lambda} \mathrm{d} \mathbf{x}}{\int \exp [-\beta \mathcal{H}(\mathbf{x}, \lambda)] \mathrm{d} \mathbf{x}} \\
& =\left\langle\frac{\partial \mathcal{H}}{\partial \lambda}\right\rangle_\lambda \\
& =\left\langle U_B-U_A\right\rangle_\lambda
\end{aligned}
$$

By definition of the integral, we naturally obtain the formula for the free energy difference:

$$
\Delta F = F_B - F_A =\int_0^1\left\langle U_B-U_A\right\rangle_\lambda \mathrm{d} \lambda
$$

In actual sampling, we use this formula by performing a series of samplings at different discrete $$\lambda$$ values and calculating the corresponding $$U_B - U_A$$, then summing them to obtain an approximate estimate of the free energy difference.

### Umbrella Sampling

From the discussion in the previous section on thermodynamic integration, we recognize that the starting point for free energy calculation is to express the change in the Hamiltonian (potential energy function) of the system either implicitly (FEP and MBAR) or explicitly (HI method) using some variable and estimate the free energy difference through sampling. With this understanding, let's look at a new sampling method (which is also a free energy calculation method): umbrella sampling, which is another explicit method for expressing Hamiltonian changes.

In umbrella sampling, we define a collective variable $$\xi(r)$$ to explicitly express the change in the system state. It is assumed that coordinates unrelated to this collective variable have a negligible effect on the system state change. Therefore, the Hamiltonian of the system becomes a function of this collective variable:

$$H = H(\mathbf{r}, \xi (\mathbf{r}))$$

Similarly, just as we sample at different parameter $$\lambda$$ values in the Hamiltonian integration method, in umbrella sampling, we sample extensively at different values of the collective variable $$\xi(r)$$. Umbrella sampling achieves this by imposing a large restraint potential on the collective variable $$\xi(r)$$:

$$
\omega_i(\xi)=K / 2\left(\xi(r)-\xi_i^{\mathrm{ref}}\right)^2
$$

The total potential energy

function with the bias potential added is:

$$
E^{\mathrm{b}}(r)=E^{\mathrm{u}}(r)+\omega_i(\xi)
$$

We know that free energy is a property of the equilibrium state of the system, meaning that free energy should be determined by the equilibrium potential energy function of the system:

$$
F_i (\xi) = - \frac{1}{\beta} \int \exp [-\beta E^{u}(r)] \delta\left[\xi(r)-\xi\right] d^{N}{r} =- \frac{1}{\beta} P_{i}^{\mathrm{u}}(\xi)
$$

where $$P_{i}^{\mathrm{u}}(\xi)$$ is the probability distribution of the collective variable $$\xi(r)$$ in the equilibrium state of the system:

$$
P_{i}^{\mathrm{u}}(\xi)=\frac{\int \exp [-\beta E(r)] \delta\left[\xi(r)-\xi\right] d^{N}{r}}{\int \exp [-\beta E(r)] d^{N}{r}}
$$

On the other hand, in umbrella sampling, the system evolves under the total potential energy function $$E^{\mathrm{b}}(r)$$ with the bias potential added. The real question in umbrella sampling is: how can we use the trajectories under the bias potential to calculate the equilibrium free energy? To achieve this, we try to use the bias potential to express the equilibrium free energy. First, we can write the probability distribution of the collective variable $$\xi(r)$$ under the bias potential:

$$ P*{i}^{\mathrm{b}}(\xi)=\frac{\int \exp \left\{-\beta\left[E(r)+\omega*{i}\left(\xi(r)\right)\right]\right\} \delta\left[\xi(r)-\xi\right] d^{N}{r}}{\int \exp \left\{-\beta\left[E(r)+\omega_{i}\left(\xi(r)\right)\right]\right\} d^{N}{r}} $$

Therefore, we can express the equilibrium probability distribution using the bias probability distribution:

$$
\begin{aligned}
P_{i}^{\mathrm{u}}(\xi)=& P_{i}^{\mathrm{b}}(\xi) \exp \left[\beta \omega_{i}(\xi)\right] \\
& \times \frac{\int \exp \left\{-\beta\left[E(r)+\omega_{i}(\xi(r))\right]\right\} d^{N}{r}}{\int \exp [-\beta E(r)] d^{N}{r}} \\
=& P_{i}^{\mathrm{b}}(\xi) \exp \left[\beta \omega_{i}(\xi)\right] \\
& \times \frac{\int \exp [-\beta E(r)] \exp \left\{-\beta \omega_{i}[\xi({r})]\right\} d^{N}{r}}{\int \exp [-\beta E(r)] d^{N}{r}} \\
=& P_{i}^{\mathrm{b}}(\xi) \exp \left[\beta \omega_{i}(\xi)\right]\left\langle\exp \left[-\beta \omega_{i}(\xi)\right]\right\rangle
\end{aligned}
$$

Note that the expectation value here is the average under the equilibrium ensemble of the system. Thus, the true equilibrium free energy of the system is:

$$
F_i (\xi) = - \frac{1}{\beta} P_{i}^{\mathrm{u}}(\xi) = -(1 / \beta) \ln P_{i}^{\mathrm{b}}(\xi)-\omega_{i}(\xi)+Q_{i}
$$

where $$Q_i = -(1 / \beta) \ln \left\langle\exp \left[-\beta \omega_{i}(\xi)\right]\right\rangle$$. We note that the true equilibrium free energy consists of three terms. The first term, $$(1 / \beta) \ln P_{i}^{\mathrm{b}}(\xi)$$, can be directly obtained from sampling. The second term, $$\omega_{i}(\xi)$$, is the known form of the bias potential. The third term, $$Q_i$$, is unknown. From its definition, we can see that $$Q_i$$ actually depends on the equilibrium distribution or the equilibrium free energy itself. Therefore, such an equation usually requires a self-consistent iterative calculation to obtain.

Notice that in the above discussion, we only considered the free energy calculation for one bias potential window. This means we only obtained the free energy $$F_i (\xi)$$ for sampling in window $$i$$. If we sample configurations only within one window, the sampled configurations will be concentrated near the center of window $$i$$, and the probability of sampling configurations outside the window will be very low, leading to a large variance in the estimated free energy. Therefore, in practice, we almost always divide the collective variable into many bins, using multiple sampling windows to sample, obtaining an estimated value of free energy over an entire interval of the collective variable.

Recall that in the BAR/MBAR method, when we have samples from multiple states, we choose appropriate parameters to minimize the variance of the free energy estimate. In umbrella sampling, we face a similar situation. The real problem in calculating free energy is: given multiple sampling points from different windows, how do we use these sampling points to minimize the variance of the free energy estimate? The Weighted Histogram Analysis Method (WHAM) aims to achieve this.

First, we set the unbiased probability distribution estimate obtained from sampling in window $$i$$ as $$P_i^{\mathrm{u}}(\xi)$$. We assume that the unbiased probability distribution $$P^{\mathrm{u}}(\xi)$$ over the entire interval of the collective variable is a linear combination of the probability distributions obtained from each window:

$$
P^{\mathrm{u}}(\xi)=\sum_i^{\text {windows }} p_i(\xi) P_i^{\mathrm{u}}(\xi)
$$

This assumption is reasonable as it satisfies that the probability ratio of different sampled configurations remains approximately unchanged within each window. To ensure that the probability distribution over the entire interval satisfies the normalization condition, we also need to satisfy:

$$
\sum p_i = 1
$$

Under this assumption, the WHAM method aims to minimize the variance of the unbiased probability distribution $$P^{\mathrm{u}}(\xi)$$ obtained by this formula. More precisely, we want to select $$p_i$$ to satisfy:

$$
\frac{\partial \sigma^2\left(P^{\mathrm{u}}\right)}{\partial p_i}=0
$$

After some tedious calculations (detailed derivation can be found in [Shankar 1992](https://quantum.ch.ntu.edu.tw/ycclab/wp-content/uploads/2015/02/WHAM_1992.pdf)), we find that the coefficients $$p_i$$ need to satisfy the following relationship:

$$
p_i=\frac{a_i}{\sum_j a_j}, a_i(\xi)=N_i \exp \left[-\beta \omega_i(\xi)+\beta F_i\right]
$$

where $$N_i$$ is the number of sampled points in window $$i$$, and the expression for $$F_i$$ is:

$$
\exp \left(-\beta F_i\right)=\int P^{\mathrm{u}}(\xi) \exp \left[-\beta w_i(\xi)\right] d \xi
$$

Notice that the expression for $$F_i$$ depends on the global probability distribution $$P^{\mathrm{u}}(\xi)$$. Similar to the single-window sampling situation, a self-consistent iterative process is still required here to finally obtain our free energy estimate.
