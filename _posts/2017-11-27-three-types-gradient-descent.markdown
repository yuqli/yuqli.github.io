---
layout: post
title:  "Three types of gradient descent"
date:   2017-11-27 16:41:34 -0400
categories: jekyll update
---

(This is a restoration of a previous post hosted on Wordpress. Hyperlinks might be missing and formatting might be a bit messy.)

Reference - Notes from this blog: https://machinelearningmastery.com/gentle-introduction-mini-batch-gradient-descent-configure-batch-size/


Three types are : batch gradient descent, stochastic gradient descent, mini-batch gradient descent

Batch gradient descent

During one epoch, evaluate error for one sample at a time. But update model only after evaluating all errors in the training set

Pros:

Calculation of prediction errors and the model update are seperated. Thus the algorithm can use parallel processing based implementations.
Cons:

Need to have all data in memory
The more stable error gradient may result in premature convergence of the model to a less optimal set of parameters.
Algo:

Model.initialize()
For i in n_epoches:
training_data.shuffle
X, Y = split(training_data)
For x in X
Y_pred = model(X) # get a vector
error = get_error(Y_pred, Y)
error_sum += error
model.update(error_sum)
Stochastic gradient descent

During one epoch, evaluate error for one sample at a time, then update model immediate after that evaluation.

Pros:

immediate feedback of model performance and improvement rate
simple to implement
frequent update -- faster learning rate
The noisy update process can allow the model to avoid local minima (e.g. premature convergence).
Cons;

frequent update - computationally extensive
add a noise parameter /  gradient signal, causing the parameters to jump around
Hard to settle to an error minimum
Algo

Model.initialize()
For i in n_epoches:
training_data.shuffle
X, Y = split(training_data)
for  each x in X
Y_pred = model(X)
error = get_error(Y_pred, Y)
model.update()
 

Mini batch gradient descent

During one epoch, split the data into batches (which adds a batch size parameter). Then for each batch, evaluate error for one sample at a time. Update the model after evaluating for all data in one batches. Repeat for different batches. Repeat for different epoch.

Pros:

more robust converge to local minima, compared to stochastic gradient descent
frequent update -- faster learning rate, compared to batch gradient descent
efficiency: no need to have all data in memory
Cons;

configuration of an additional mini-batch parameter
Algo

Model.initialize()
For i in n_epoches:
training_data.shuffle
Data.split.batches
For j in n_batches
X, Y = split(batch_data)
For x in X:
Y_pred = model(X) # here a vector
error = get_error(Y_pred, Y)
error_sum += error
model.update()
