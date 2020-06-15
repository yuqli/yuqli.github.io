---
layout: post
title:  "Really understand VAE"
date:   2019-01-23 17:36:34 -0400
categories: jekyll update
---
(This is a restoration of a previous post hosted on Wordpress.)

It took me a long while (~ 4 days?) to understand the theories of Variation autoencoder (VAE) and how to actually implement it. And it's not entirely my fault, because:

The original paper (Auto-Encoding Variational Bayes) explains things from a probabilistic perspective, which
requires knowledge on Bayesian statistics to know the math is correct
requires knowledge on variational inference, or general statistical inference to fully understand motivations of introducing distributional assumptions, and meanings of equations. (In fact, I should be familiar with this because I did something related in my final year of college, but I forgot most of it..)
only offers a general and abstract framework for the major parts of the paper, but leaves results for specific distributions in the appendix.
Online tutorials over-simplify this model using neural net languages
Most online tutorials just add a "sampling random noise from normal distribution" step in the auto-encoder framework. But often these tutorials cannot hold up in their writing and have logic loopholes. e.g. they never explain why the loss term have a KL-divergence between sampled and true z distributions, and how to calculate that distribution using empirical data (this is a key).
Most online implementations miss a point: if reconstructed x is sampled, how can we use MSE loss between original data and reconstructed data? They are not even the same data point. Some tries to explain this by acknowledging the reconstructed x is sampled from conditional P(x|z_i) and thus establishing the correspondence, but still, reconstructed x still has a randomness in it, and cannot use MSE loss.
Even the lecture on VAE from Stanford  does not explicitly explain this point above.
One will feel the online tutorial and the original paper do not match up.
In other words, most online tutorials do not bridge the gap from the general variational lower bound to the actual implementation equation, because they do not introduce three crucial distributional assumptions.

In fact, one just cannot derive the loss function in the original paper from a neural net perspective, because neural net does not make distributional assumptions. Also, the original paper is not about neural networks.

So, this post just tries point out the key ideas in understanding the original paper:

The whole paper sets in the probabilistic framework. We do not have loss functions here. Instead, we make an assumption for p(x), and the goal is to find parameters for p to maximize likelihood to account for the current observed data x. Once we find these parameters, we can sample new data from this distribution.
We introduce a latent variable z in order to better estimate p(x)
We don't know the conditional distribution of z|x. So we assume it follows normal, i.e. q(z|x)
After introducing q(z|x), while we still can't analytically tackle p(x), we can at least get a lower bound, which is - D_{KL} (q(z|x_i) || p(z)) + E_q ( log p(x|z)) You can find how to get this lower bound from many online tutorials, e.g. this.
With the important lower bound we got from 4, now it's time to plug in some concrete distributional assumptions. In the original paper appendix B, an analytical solution for - D_{KL} (q(z|x_i) || p(z)) is given when p(z) and q(x|z) are both Gaussian
But then still we need to get $latex E_q ( log p(x|z))$. This is simplified to l2 loss when x is assumed to follow i.i.d. Gaussian. To see why, think about linear regression and how MLE is the same as minimizing the total sum of squares in terms of getting estimates for regression coefficients. Also this slide from UIUC!
So now we got a concrete instance for VAE under Gaussian assumptions. How does this apply to neural network and autoencoders? Here is a glossary:
encoder: output z|x, and the neural net is to simulate sampling from p(z|x). Used to get empirical observations of z. we have a loss term here in that we want to minimize the KL divergence between this output z|x and our assumption that z|x follows normal.
decoder: output x|z. the neural net is to simulate sampling from p(x|z)
