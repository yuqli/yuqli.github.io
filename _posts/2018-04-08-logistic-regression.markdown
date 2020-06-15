---
layout: post
title:  "Two approaches for logistic regression"
date:   2018-04-08 19:42:34 -0400
categories: jekyll update
---

(This is a restoration of a previous post hosted on Wordpress. Hyperlinks might be missing and formatting might be a bit messy.)

Finally, a post in this blog that actually gets a little bit technical ...

This post discusses two approaches for understanding of logistic regression: Empirical risk minimizer vs probabilistic approaches.

Empirical Risk Minimizer

Empirical risk minimizing frames a problem in terms of the following components:

Input space X \in R^d. Corresponds to observations, features, predictors, etc
outcome space Y \in \Omega. Corresponds to target variables.
Action space A_R  = R  Also called decision function, predictor, hypothesis.
A sub element in action space could be hypothesis space: all linear transformations
Loss function: l(\hat{y}, y) a loss defined on the predicted values and observed values.
The goal of the whole problem is to select a function mapping $F$ in action space that minimizes the total loss on sample. This is achieved by selecting the value of the parameters in $f$ such that it minimizes the empirical loss in the training set. We also do hyperparameter tuning, which is done on the validation set in order to prevent overfitting.

In a binary classification task:

Input space X \in R^d.
outcome space Y \in {0, 1}. Binary target values.
Action space A_R  = R  The hypothesis space: all linear score functions
F_{score} = {x \rightarrow x^Tw | w \in R^d}
Loss function: l(\hat{y}, y) = l_{logistic}(m) = \text{log} (1 + e ^{-m})
This is a kind of margin based loss, thus the m here.
Margin is defined as \hat{y} y, which has interpretation in binary classification task. Consider:
if m = \hat{y} y > 0 , we know we have our prediction and true value are of the same sign. Thus, in binary classification, we could already get the correct result. Thus, for m > 0 we should have loss = 0.
if m = \hat{y} y < 0 , we know we have our prediction and true value are of different signs. Thus, in binary classification, we are wrong. We need to define a positive value for loss function.
In SVM, we define hinge loss l(m) = \text{max}(0, 1-m), which is a "maximum-margin" based loss (more on this in the next post, which will cover the derivation of SVM, kernel methods) Basically, for this loss, we have when m \geq 1 no loss, $\latex m < 1$ loss. We can interpret m as "confidence" of our prediction. When m < 1 this means a low confidence, thus still penalize!
With this intuition, how do we understand logistic loss? We know:
This loss always > 1
When m negative (i.e. wrong prediction), we have greater loss !
When m positive (i.e. correct prediction), we have less loss...
Note also for same amount of increase in m, the scale that we "reward" correct prediction is less than the scale we penalize wrong predictions.
Bournoulli regression with logistic transfer function

Input space X = R^d
Outcome space y \in {0, 1}
action space A = [0, 1] An action is the probability that an outcome is 1
Define the standard logistic function as \phi (\eta) = 1 / (1 + e^{-\eta})

Hypothesis space as F = {x \leftarrow\phi (w^Tx) | w \in R^d}
Sigmoid function is any function that has an "S" shape. One example is the simple case of logistic function! Used in neural networks as activation function / transfer function. Purpose is to add non-linearity to the network.
Now we need to do a re-labeling for y_i in the dataset.

For every y_i = 1, we define y'  = 1
For every y_i = -1, we define y'  = 0
Can we do this? Doesn't this change the value of y-s ? The answer is , in binary classification ( or in any classification), the labels do not matter. Instead, this trick just makes the equivalent shown much easier...

Then, the negative log likelihood objective function, given this F and dataset $D$, is :

NLL(w) = \Sum^n [-y_i ' \text{log} \phi (w^T x_i)] +(y_i ' -1) \text{log} (1 - \phi (w^T x_i))
How to understand this approach? Think about a neural network...

Input x
First linear layer: transform x into w^Tx
Next non-linear activation function. \phi (\eta) = 1 / (1 + e^{-\eta}).
The output is interpreted as a probability of positive classes.
Think about multi-class problems, the second layer is a softmax --- and we get a vector of probabilities!
With some calculation, we can show NLL is equivalent to the sum of empirical loss.

 