---
layout: post
title: Diffusion model in SO(3) space
date: 2023-08-01 0:00:00
description: this is what a post that can be cited looks like
tags: formatting citation
categories: sample-posts
citation: true
---

This theme supports rendering beautiful math in inline and display modes using [MathJax 3](https://www.mathjax.org/) engine. 

# Why bother diffusion in SO(3) space?
One typical way to represent the structure of proteins is to use the position of alpha carbon and the rotation matrix of each residue. So the diffusion process on the space of alpha carbon is simple Euclidean space, but the diffusion process on the rotation matrix ($$SO(3)$$ space) is non-Euclidean. This note aims to provide a rigorous formulation of diffusion model on $$SO(3)$$ space.

# Some definitions and identities
In this note, we do not always distinguish "group" and "representation of group", just like physicists.

With notation from the article, we define the basis of the lie algebra of $$SO(3)$$ group and their matrix representation as follows:
\begin{equation}
\begin{array}{ll}
\mathbf{e}_1=\left[\begin{array}{l}
1 \\
0 \\
0 \\
\end{array}\right] & G_1=\mathbf{e}_1^{\times}=\left[\begin{array}{ccc}
0 & 0 & 0 \\
0 & 0 & -1 \\
0 & 1 & 0
\end{array}\right] 
\\
\\
\mathbf{e}_2=\left[\begin{array}{l}
0 \\
1 \\
0
\end{array}\right] & G_2=\mathbf{e}_2^{\times}=\left[\begin{array}{ccc}
0 & 0 & 1 \\
0 & 0 & 0 \\
-1 & 0 & 0
\end{array}\right]
\\
\\
\mathbf{e}_3=\left[\begin{array}{l}
0 \\
0 \\
1
\end{array}\right] & G_3=\mathbf{e}_3^{\times}=\left[\begin{array}{ccc}
0 & -1 & 0 \\
1 & 0 & 0 \\
0 & 0 & 0
\end{array}\right]
\end{array}
\end{equation}

Thus we can exponentiate the lie algebra matrix representation to get the matrix representation of $SO(3)$ group, and get back by taking the logarithm of $$SO(3)$$ matrix.

The "box plus" operation is defined between a $$SO(3)$$ matrix $$\Phi$$ and a lie algebra vector $$\varphi$$
\begin{equation}
\label{eq:0}
\boxplus: SO(3) \times \mathbb{R}^3 \rightarrow SO(3), \quad
(\Phi, \boldsymbol{\varphi}) \mapsto \exp (\boldsymbol{\varphi}^{\times}) \circ \Phi
\end{equation}

The "box minus" operation is defined between two $$SO(3)$$ matrix.
\begin{equation}
\boxminus: SO(3) \times SO(3) \rightarrow \mathbb{R}^3, \quad
(\Phi_1, \Phi_2) \mapsto \log \left(\Phi_1 \circ \Phi_2^{-1}\right)
\end{equation}

\textbf{Some important identities} ($$\boldsymbol{v}$$ is a vector in lie algebra, $$\boldsymbol{v}^{\times}$$ is its matrix representation):
\begin{equation}
\label{eq:1}
\begin{aligned}
\left(\boldsymbol{v}^{\times}\right)^T & =-\boldsymbol{v}^{\times}, \\
\left(\boldsymbol{v}^{\times}\right)^2 & =\boldsymbol{v} \boldsymbol{v}^T-\boldsymbol{v}^T \boldsymbol{v} \boldsymbol{I}, \\
(\Phi \boldsymbol{v})^{\times} & = \Phi \boldsymbol{v}^{\times} \Phi^T
\end{aligned}
\end{equation}
Especially the third identities, which will play important roles in the derivation of the score function.

With these definitions, we can define the derivative of a scalar function with argument to be $$SO(3)$$ matrix,
\begin{equation}
\label{eq:2}
\frac{\partial f_3}{\partial \Phi}=\lim _{\epsilon \rightarrow 0}\left[\begin{array}{l}
\frac{f_3\left(\Phi \boxplus\left(\mathbf{e}_1 \epsilon\right)\right)-f_3(\Phi)}{\epsilon} \\
\frac{f_3\left(\Phi \boxplus\left(\mathbf{e}_2 \epsilon\right)\right)-f_3(\Phi)}{\epsilon} \\
\frac{f_3\left(\Phi \boxplus\left(\mathbf{e}_3 \epsilon\right)\right)-f_3(\Phi)}{\epsilon}
\end{array}\right]
\end{equation}

We can represent the $$SO(3)$$ matrix in 3D space using axis-angle representation, in explicit terms, it can be written as:
\begin{equation}
\overleftrightarrow{R}_n(\theta)_{i, j}=n_i n_j+\cos \theta \cdot\left(\delta_{i, j}-n_i n_j\right)-\sin \theta \cdot \sum_c \epsilon_{i j k} n_k, \text { here } i, j, k=x, y, z
\end{equation}
Or more simply written using lie algebra matrix:
\begin{equation}
\overleftrightarrow{R}_n(\theta)=\exp ( \theta \overleftrightarrow{G} \bullet n)
\end{equation}
where $$\theta$$ is in the range $$[0, \pi]$$, and $$n$$ represents the rotation axis.




Note that MathJax 3 is [a major re-write of MathJax](https://docs.mathjax.org/en/latest/upgrading/whats-new-3.0.html) that brought a significant improvement to the loading and rendering speed, which is now [on par with KaTeX](http://www.intmath.com/cg5/katex-mathjax-comparison.php).
