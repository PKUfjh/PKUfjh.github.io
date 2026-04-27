---
layout: post
title: Guided diffusion models
description: This post describes diffusion models with guidance.
date: 2024-07-03 0:00:00
tags: AI
---

# Motivations

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/guided-diffusion.png" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Generation of dogs through guided diffusion, with increasing amounts of guidance from left to right.
</div>

Just as we generate wanted dogs pictures with learned guidance in the above figure, in generating molecule structures we also want to incorporate the physical knowledge, in order to generate structures whose distribution are closer to the true Boltzmann distribution. This can be done by the "energy-guided diffusion models" which will be introduced later.

# Diffusion model review

Forward process:

$$
\begin{equation}
q\left(\mathbf{x}_t \mid \mathbf{x}_{t-1}\right)=\mathcal{N}\left(\mathbf{x}_t ; \sqrt{1-\beta_t} \mathbf{x}_{t-1}, \beta_t \mathbf{I}\right) \quad q\left(\mathbf{x}_{1: T} \mid \mathbf{x}_0\right)=\prod_{t=1}^T q\left(\mathbf{x}_t \mid \mathbf{x}_{t-1}\right)
\end{equation}
$$

Backward process:

$$
\begin{equation}
q\left(\mathbf{x}_{t-1} \mid \mathbf{x}_t, \mathbf{x}_0\right)=q\left(\mathbf{x}_t \mid \mathbf{x}_{t-1}, \mathbf{x}_0\right) \frac{q\left(\mathbf{x}_{t-1} \mid \mathbf{x}_0\right)}{q\left(\mathbf{x}_t \mid \mathbf{x}_0\right)}
\end{equation}
$$

The forward process is a determined process in diffusion models, it is only a matter of learning the conditional probability function $$q\left(\mathbf{x}_{t-1} \mid \mathbf{x}_t, \mathbf{x}_0\right)$$ or the so-called score function $$\nabla_{x_{t-1}} \log q\left(\mathbf{x}_{t-1} \mid \mathbf{x}_t, \mathbf{x}_0\right)$$ in order to learn the data distribution and to do the sampling.

# Classifier guidance

In the framework of classifier guidance, a classifier function should be learned to be used as guidance, namely the new score function goes like:

$$
\begin{equation}
\nabla_{x_t} \log p(x_t | y) =  \nabla_{x_t} \log p(x_t) + \nabla_{x_t} \log p(y | x_t)
\end{equation}
$$

where $$\nabla_{x_t} \log p(x_t)$$ is the original score function, $$\nabla_{x_t} \log p(y \mid x_t)$$ is the learned classifier function term.

In order to derive this formulation, A naive derivation goes like this:

$$
\begin{equation}
    p(x_t | y) = \frac{p(x_t, y)}{p(y)} = \frac{p(y | x_t) p(x_t)}{p(y)}
\end{equation}
$$

where the new probability function now depends on the guidance label $$y$$. And since the denominator does not depend on $$x_t$$, we can reach the above formulation after taking logarithm and derivative.

# Classifier-free guidance

Let's add the scale factor to the classifier guidance score function:

$$
\begin{equation}
 \nabla_{x_t} \log p(x_t) + \eta \nabla_{x_t} \log p(y | x_t)
\end{equation}
$$

Now we again use the bayes' rule:

$$
\begin{equation}
 p(y | x_t) =  p(x_t | y) \frac{p(y)}{p(x_t)}
\end{equation}
$$

So the score function can be rewritten as:

$$
\begin{equation}
\begin{aligned}
s(x_t, x_{t+1}, y) & =  \nabla_{x_t} \log p(x_t) + \eta \nabla_{x_t} (\log p(x_t | y) -\log p(x_t) )\\
& = (1-\eta)\nabla_{x_t} \log p(x_t) + \eta \nabla_{x_t} \log p(x_t | y)
\end{aligned}
\end{equation}
$$

Namely, we can train one model conditioned on the label selectively. When doing inference, we can use $$\eta$$ to control the scale of the conditioned score function.

# Energy guided diffusion

The target distribution is:

$$
\begin{equation}
p_0\left(\boldsymbol{x}_0\right) \propto q_0\left(\boldsymbol{x}_0\right) e^{-\beta \mathcal{E}\left(\boldsymbol{x}_0\right)} .
\end{equation}
$$

Suppose the marginal distribution $$q_t (x_t)$$ of the forward process starting from $$q_0(x_0)$$, then a pre-trained model can be obtained by learning the the score function $$\nabla_{x_t} \log q_t(x_t)$$. In the same flavor, in order to learn $$p_0 (x_0)$$, the model have to learn the intermediate score function $$\nabla_{x_t} \log p_t(x_t)$$. In the following we show that under the assumption that the noising process for $$p_0(x_0)$$ and $$q_0(x_0)$$ are the same, the noisy distribution $$p_t(x_t)$$ and $$q_t(x_t)$$ will satisfy:

$$
\begin{equation}
p_t\left(\boldsymbol{x}_t\right) \propto q_t\left(\boldsymbol{x}_t\right) e^{-\mathcal{E}_t\left(\boldsymbol{x}_t\right)},
\end{equation}
$$

where：

$$
\begin{equation}
\mathcal{E}_t\left(\boldsymbol{x}_t\right):= \begin{cases}\beta \mathcal{E}\left(\boldsymbol{x}_0\right), & t=0 \\ -\log \mathbb{E}_{q_{0 t}\left(\boldsymbol{x}_0 \mid \boldsymbol{x}_t\right)}\left[e^{-\beta \mathcal{E}\left(\boldsymbol{x}_0\right)}\right], & t>0\end{cases}
\end{equation}
$$

We give a proof here:

$$
\begin{equation}
p_{t 0}\left(\boldsymbol{x}_t \mid \boldsymbol{x}_0\right):=q_{t 0}\left(\boldsymbol{x}_t \mid \boldsymbol{x}_0\right)=\mathcal{N}\left(\boldsymbol{x}_t \mid \alpha_t \boldsymbol{x}_0, \sigma_t^2 \boldsymbol{I}\right)
\end{equation}
$$

$$
\begin{equation}
\begin{aligned}
p_t\left(\boldsymbol{x}_t\right) & =\int p_{t 0}\left(\boldsymbol{x}_t \mid \boldsymbol{x}_0\right) p_0\left(\boldsymbol{x}_0\right) \mathrm{d} \boldsymbol{x}_0 \\
& =\int p_{t 0}\left(\boldsymbol{x}_t \mid \boldsymbol{x}_0\right) q_0\left(\boldsymbol{x}_0\right) \frac{e^{-\beta \mathcal{E}\left(\boldsymbol{x}_0\right)}}{Z} \mathrm{~d} \boldsymbol{x}_0 \\
& =\int q_{t 0}\left(\boldsymbol{x}_t \mid \boldsymbol{x}_0\right) q_0\left(\boldsymbol{x}_0\right) \frac{e^{-\beta \mathcal{E}\left(\boldsymbol{x}_0\right)}}{Z} \mathrm{~d} \boldsymbol{x}_0 \\
& =q_t\left(\boldsymbol{x}_t\right) \int q_{0t}\left(\boldsymbol{x}_0 \mid \boldsymbol{x}_t\right) \frac{e^{-\beta \mathcal{E}\left(\boldsymbol{x}_0\right)}}{Z} \mathrm{~d} \boldsymbol{x}_0 \\
& =\frac{q_t\left(\boldsymbol{x}_t\right) \mathbb{E}_{q_{0t}\left(\boldsymbol{x}_0 \mid \boldsymbol{x}_t\right)}\left[e^{\left.-\beta \mathcal{E}\left(\boldsymbol{x}_0\right)\right]}\right.}{Z} \\
& =\frac{q_t\left(\boldsymbol{x}_t\right) e^{-\mathcal{E}_t\left(\boldsymbol{x}_t\right)}}{Z}
\end{aligned}
\end{equation}
$$

The score function with energy guidance is:

$$
\begin{equation}
\nabla_{\boldsymbol{x}_t} \log p_t\left(\boldsymbol{x}_t\right)=\underbrace{\nabla_{\boldsymbol{x}_t} \log q_t\left(\boldsymbol{x}_t\right)}_{\approx-\boldsymbol{\epsilon}_\theta\left(\boldsymbol{x}_t, t\right) / \sigma_t}-\underbrace{\nabla_{\boldsymbol{x}_t} \mathcal{E}_t\left(\boldsymbol{x}_t\right)}_{\begin{array}{c}
\text {energy guidance (intractable)}
\end{array}} .
\end{equation}
$$

The first term is given by the pretrained model, the remaining problem is to estimate the latter. We show in the next slide that the solution $$f_{\boldsymbol{\phi}^*}\left(\boldsymbol{x}_t, t\right)$$ to the following problem:

$$
\begin{equation}
\min _\phi \mathbb{E}_{p(t)} \mathbb{E}_{q_0\left(\boldsymbol{x}_0^{(1: K)}\right)} \mathbb{E}_{p\left(\boldsymbol{\epsilon}^{(1: K)}\right)}\left[-\sum_{i=1}^K e^{-\beta \mathcal{E}\left(\boldsymbol{x}_0^{(i)}\right)} \log \frac{e^{-f_\phi\left(\boldsymbol{x}_t^{(i)}, t\right)}}{\sum_{j=1}^K e^{-f_\phi\left(\boldsymbol{x}_t^{(j)}, t\right)}}\right]
\end{equation}
$$

will satisfy $$\nabla_{\boldsymbol{x}_t} f_{\boldsymbol{\phi}^*}\left(\boldsymbol{x}_t, t\right)=\nabla_{\boldsymbol{x}_t} \mathcal{E}_t\left(\boldsymbol{x}_t\right)$$. Intuitively, it is the cross-entropy loss between the intermediate energy model and the trained model.

# Force guided diffusion

Similar to the intermediate energy expression, we can define the intermediate force as:

$$
\begin{equation}
\nabla_{\mathbf{x}_t} \mathcal{E}_t\left(\mathbf{x}_t\right)=\frac{\mathbb{E}_{q_t\left(\mathbf{x}_0 \mid \mathbf{x}_t\right)}\left[e^{-k \mathcal{E}_0\left(\mathbf{x}_0\right)} \zeta\left(\mathbf{x}_0, \mathbf{x}_t\right)\right]}{k \mathbb{E}_{q_t\left(\mathbf{x}_0 \mid \mathbf{x}_t\right)}\left[e^{-k \mathcal{E}_0\left(\mathbf{x}_0\right)}\right]}
\end{equation}
$$

where $$\zeta\left(\mathbf{x}_0, \mathbf{x}_t\right)=\nabla_{\mathbf{x}_t} \log q_t\left(\mathbf{x}_t\right)-\nabla_{\mathbf{x}_t} \log q_t\left(\mathbf{x}_t \mid \mathbf{x}_0\right)$$.
We give a proof here, from the expression of the intermediate energy, we have the expression of the intermediate force

$$
\begin{equation}
\nabla_{\mathbf{x}_t} \mathcal{E}_t\left(\mathbf{x}_t\right)=-\frac{\int e^{-k \mathcal{E}_0\left(\mathbf{x}_0\right)} \nabla_{\mathbf{x}_t} q_t\left(\mathbf{x}_0 \mid \mathbf{x}_t\right) \mathrm{d} \mathbf{x}_0}{k \int q_t\left(\mathbf{x}_0 \mid \mathbf{x}_t\right) e^{-k \mathcal{E}_0\left(\mathbf{x}_0\right)} \mathrm{d} \mathbf{x}_0}
\end{equation}
$$

The numerator $$N$$ evaluates as:

$$
\begin{equation}
\begin{aligned}
N & =\int e^{-k \mathcal{E}_0\left(\mathbf{x}_0\right)} q_t\left(\mathbf{x}_0 \mid \mathbf{x}_t\right) \nabla_{\mathbf{x}_t} \log q_t\left(\mathbf{x}_0 \mid \mathbf{x}_t\right) \mathrm{d} \mathbf{x}_0 \\
& =\int q_t\left(\mathbf{x}_0 \mid \mathbf{x}_t\right) e^{-k \mathcal{E}_0\left(\mathbf{x}_0\right)} \nabla_{\mathbf{x}_t} \log \frac{q_t\left(\mathbf{x}_t \mid \mathbf{x}_0\right) q_0\left(\mathbf{x}_0\right)}{q_t\left(\mathbf{x}_t\right)} \mathrm{d} \mathbf{x}_0 \\
& =\int q_t\left(\mathbf{x}_0 \mid \mathbf{x}_t\right) e^{-k \mathcal{E}_0\left(\mathbf{x}_0\right)}\left(\nabla_{\mathbf{x}_t} \log q_t\left(\mathbf{x}_t \mid \mathbf{x}_0\right)-\nabla_{\mathbf{x}_t} \log q_t\left(\mathbf{x}_t\right)\right) \mathrm{d} \mathbf{x}_0 \\
& =\mathbb{E}_{q_t\left(\mathbf{x}_0 \mid \mathbf{x}_t\right)}\left[e^{-k \mathcal{E}_0\left(\mathbf{x}_0\right)}\left(\nabla_{\mathbf{x}_t} \log q_t\left(\mathbf{x}_t \mid \mathbf{x}_0\right)-\nabla_{\mathbf{x}_t} \log q_t\left(\mathbf{x}_t\right)\right)\right]
\end{aligned}
\end{equation}
$$

the intermediate force evaluates as:

$$
\begin{equation}
\begin{aligned}
\nabla_{\mathbf{x}_t} \mathcal{E}_t\left(\mathbf{x}_t\right) & =-\frac{\mathbb{E}_{q_t\left(\mathbf{x}_0 \mid \mathbf{x}_t\right)}\left[-e^{-k \mathcal{E}_0\left(\mathbf{x}_0\right)} \zeta\left(\mathbf{x}_0, \mathbf{x}_t\right)\right]}{k \mathbb{E}_{q_t\left(\mathbf{x}_0 \mid \mathbf{x}_t\right)}\left[e^{-k \mathcal{E}_0\left(\mathbf{x}_0\right)}\right]} \\
& =\frac{\mathbb{E}_{q_0\left(\mathbf{x}_0\right)}\left[q_t\left(\mathbf{x}_t \mid \mathbf{x}_0\right) e^{-k \mathcal{E}_0\left(\mathbf{x}_0\right)} \zeta\left(\mathbf{x}_0, \mathbf{x}_t\right)\right]}{k \mathbb{E}_{q_0\left(\mathbf{x}_0\right)}\left[q_t\left(\mathbf{x}_t \mid \mathbf{x}_0\right) e^{-k \mathcal{E}_0\left(\mathbf{x}_0\right)}\right]} .
\end{aligned}
\end{equation}
$$

where $$\zeta\left(\mathbf{x}_0, \mathbf{x}_t\right)=\nabla_{\mathbf{x}_t} \log q_t\left(\mathbf{x}_t\right)-\nabla_{\mathbf{x}_t} \log q_t\left(\mathbf{x}_t \mid \mathbf{x}_0\right)$$.
So the idea is that we can train a model to learn this intermediate force, and apply that to guide the diffusion process, namely the score function goes:

$$
\begin{equation}
\nabla_{\mathbf{x}_t} \log p_t\left(\mathbf{x}_t\right)=\nabla_{\mathbf{x}_t} \log q_t\left(\mathbf{x}_t\right)-\eta \nabla_{\mathbf{x}_t} \mathcal{E}_t\left(\mathbf{x}_t\right),
\end{equation}
$$

where $$\eta$$ is the parameter which controls the guidance strength.

Finally we have the force-guided algorithm:

$$
\begin{equation}
\begin{aligned}
&\text { Input: generated data } \mathbf{x}_0 \text { in a batch } K \text {, score model }\\
&s_\theta\left(\mathbf{x}_t, t\right) \text { in Section 4.1, energy } \mathcal{E}_0\left(\mathbf{x}_0\right) \text {, force } \nabla_{\mathbf{x}_0} \mathcal{E}_0\left(\mathbf{x}_0\right) \text {, }\\
&\text { intermediate force network } h_\psi\left(\mathbf{x}_t, t\right)\\
&\text { for training iterations do }\\
&t \sim \mathcal{U}(0,1) .\\
&\mathbf{x}_t=\text { Forward diffusion}\\
&q_t\left(\mathbf{x}_t \mid \mathbf{x}_0\right) \sim \mathcal{N}\left(\mathbf{x}_t ; \sqrt{\alpha_t} \mathbf{x}_0,\left(1-\alpha_t\right) I\right)\\
&\zeta\left(\mathbf{x}_0, \mathbf{x}_t\right)=s_\theta\left(\mathbf{x}_t, t\right)-\nabla \log q_t\left(\mathbf{x}_t \mid \mathbf{x}_0\right) \text {. }\\
&Y=\sum_{\mathbf{x}_0} q_t\left(\mathbf{x}_t \mid \mathbf{x}_0\right) e^{-k \mathcal{E}_0\left(\mathbf{x}_0\right)} \text {. }\\
&\mathcal{L}=\frac{1}{K} \sum\left\|h_\psi\left(\mathbf{x}_t, t\right)-e^{-k \mathcal{E}_0\left(\mathbf{x}_0\right)} \zeta\left(\mathbf{x}_0, \mathbf{x}_t\right) / Y\right\|_2^2 .\\
&\min _\psi \mathcal{L} \text {. } \text{end for}.
\end{aligned}
\end{equation}
$$
