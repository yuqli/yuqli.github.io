---
layout: post
title:  "Various SGDs"
date:   2019-01-25 20:54:34 -0400
categories: jekyll update
---
(This is a restoration of a previous post hosted on Wordpress. Hyperlinks might be missing and formatting might be a bit messy.)

This post briefly documents variations of optimization algorithms and best practices. It's a summary of this source . Also this course note is helpful for a review.

Tricks in checking if a gradient is implemented correctly: use centered difference instead of finite difference, because the first one has an error of O^2 if you try expand it by Taylor's theorem. It's more precise.

Momentum 



Interpretation: the momentum factor is multiplied on the past update velocity and decreases it. Momentum is more like a friction of the previous update scale. Also plus the newly learnt gradient. As a result if gradient goes to another direction it will be slowed down. If gradient goes to the same direction the scale of change will become faster and faster.

Nesterov Momentum

Similar to Momentum, but the update step, instead of calculating gradient at J(theta), we calculate it as theta + r * v, as a "look ahead step". we know our momentum factor will take us to a different location and we get the gradient from that location.

Exponential decay of learning rate

As learning approaching the end, it makes sense to decrease the learning rate and go in smaller steps.

Adagrad

cache += dx ** 2

x += lr * dx / (sqrt(cache) + eps)

eps is a smoothing factor to avoid division by zero

Understanding: 1) the scaling factor is square root of sums for all previous gradients but in a per dimension fashion so that every dimension has its own learning rate 2) as a result, for more sparse parameters the learning rate is increased because the sum of gradients are smaller, and for dense parameters the learning rate is decreased because the sum of gradients are larger

RMSprop

It's an adjustment to Adam in that the learning factor no longer goes monotonically decreasing (because the cache is always increasing). Instead, the cache here is "leaky" think of LSTM gates. The formula is :

cache = decay_rate * cache + (1 - decay_rate) * dx**2
x += - learning_rate * dx / (np.sqrt(cache) + eps)
Question: what's the initial cache value for RMSprop? some implementation states it's zero. 

Adam

Adam looks like a mixture of RMSprop and momentum.

m = beta1 * m + (1 - beta1) * dx

v = beta2 * v + (1 - beta2) * (dx**2)

x += -lr * m / (sqrt(v) + eps)

Note here 1) the m and v are leaky just as in RMSprop and 2) the update step is also exactly like RMSprop except that we now us a "smooth" version m instead of the raw gradient dx and 3) the construction of m looks like momentum.

