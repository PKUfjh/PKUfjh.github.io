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

where $$\nabla_{x_t} \log p(x_t)$$ is the original score function, $$\nabla_{x_t} \log p(y | x_t)$$ is the learned classifier function term.

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

Namely, we can train one model conditioned on the label selectively. When doing inference, we can use $\eta$ to control the scale of the conditioned score function.
