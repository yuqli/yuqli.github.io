---
layout: post
title:  "DCGAN Summary and Experiments"
date:   2019-02-05 21:56:34 -0400
categories: jekyll update
---
(This is a restoration of a previous post hosted on Wordpress. Hyperlinks might be missing and formatting might be a bit messy.)

This post 1) summarizes the paper "unsuperverized representation learning with deep convolutional generative adversarial networks (DCGAN)" as well as my empirical experiment results.

# Key idea 
This paper teaches a family of CNNs combined with GAN for (better) unsupervised learning. The trained discriminator can be used for image classification tasks, and the filters learnt by GAN can learn to draw specific objects. The generators also have interesting vector arithmetic properties.

# Background 
Although theoretically CNN can be combined with GAN for generative models, in practice it's hard to do so because GANs are hard to train. Thus, these authors propose certain architecture constraints (after many failed experiments probably).

This paper is based on three piles of literature: representation learning from unlabelled data, generating natural images, and visualizing cnns.

# Thesis
- Replace any pooling layer with strided convolutions for discriminator, and use fractional-strided convolutions for generator. This way the network can learn its own spatial downsampling... why?
- eliminating all fully connected layers on top of convolutional features. Even for the last layer, the convolution layer is flattened and fed into a single sigmoid output.
- batch normalization is applied to applied to every layer except for generator output and discriminator input layer.
- ReLU is used in the generator except for the output layer which uses Tanh, because bounded activation allowed the model to learn more quickly to saturate.
- use LeakyReLU activation in the discriminator for all layers.

# Trivia 
- the faster models learn, the better their generalization performances are Hardt et al 2015
- use the center of images for deduplication
- openCV has a face detector that can extract faces
- fractionally-strided convolutions are wrongly called deconvolutions --- check that Twitter paper that argues they are the same thing

# Experiments
- Using the features learnt by generator for classification tasks. Performs better than all k-means extracted features
- visualizing the latent space by sampling points and interpolate between two points.
- Visualize effects of different filters and vector arithmetics
