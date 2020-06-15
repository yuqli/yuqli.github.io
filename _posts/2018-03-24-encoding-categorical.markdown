---
layout: post
title:  "Encoding categorical features: likelihood, one-hot, and feature selection"
date:   2018-03-24 01:07:34 -0400
categories: jekyll update
---

(This is a restoration of a previous post hosted on Wordpress. Hyperlinks might be missing and formatting might be a bit messy.)

This post describes techniques used to encode high cardinality categorical features in a supervised learning problem.

In particular, since these values cannot be ordered, the features are nominal. Specifically, I am working with the Kaggle competition here. The problem with this dataset is that some features (e.g. types of cell phone operating systems) are categorical and has hundreds of values.

The problem occurs in how to fit these features in our model. Nominal features work fine with decision trees (random forests), Naive Bayes (use count to estimate pmf). But for other models, e.g. neural networks, logistic regression, the input needs to be numbers.

Before introducing likelihood encoding, we can go over other methods in handling such situations.

Likelihood encoding 

Likelihood encoding is a way of representing the values according to their relationships with the target variable. The goal is finding a meaningful numeric encoding for a categorical feature. Meaningful in this case means as much related to the output/target as possible.

How do we do this? A simple way is 1) first group the training set by this particular categorical feature and 2) representing each value by within group mean of that value. For example, a categorical feature might be gender. Suppose the target is height. Then, we might have the average height for male is 1.70m, while the average height for female is 1.60m. We then change 'male' to 1.70, while 'female' to 1.60.

Perhaps we should also add some noise to this mean to prevent overfitting to training data. This can be done by :

add Gaussian noise to the mean. Credit to Owen Zhang :
use the idea of "cross-validation". So here, instead of using the grand group mean, we use the cross-validation mean. (Not very clear on this point at the moment. Need to examine the idea of cross-validation. Will write in the next post.) Some people propose on Kaggle about using two levels of cross-validation: https://www.kaggle.com/tnarik/likelihood-encoding-of-categorical-features 
One hot vector

This idea is similar to the dummy variable in statistics. Basically, each possible value is being transformed into its own columns. Each of these columns will be a 1 if the original feature equals this value, or 0 if the original feature does not equal this value.

An example is for natural language processing models, the first step is usually 1) tokenize the sentence and 2) constructing a vocabulary and 3) map every token to an index (a number, or a nominal value, basically). After that, we do 4) one hot encoding and 5) a linear transformation in the form of a linear layer in a neural network (basically transform high-dim one hot

After that, we do 4) one hot encoding and 5) a linear transformation in the form of a linear layer in a neural network (basically transform high-dim one hot vectors into low dim vectors). In this way, we are basically representing every symbol in a low dimensional vector. The exact form of the vector is learned. What is happening here is actually dimension reduction.  So, after learning the weighting matrix, other methods, like PCA, can potentially work here as well!

Hashing 

A classmate named Lee Tanenbaum told me about this idea. This is an extension on the one-hot encoding idea. Suppose there can be n values in the feature. Basically, we use two hash functions, hash the possible values into two variables. The first hash all values into \sqrt(n) number of baskets,basketh baskent there are \sqrt(n) number of feature values. All feature values in the same busket is going to be the same for variable A. Then, we use a second hash function, that carefully hash the values into another busket variable B. We want to make sure the combination of A and B can fully represent every possible value in the target feature. We then learn a low-dim representation for both A and B, and concantenate them together.

My opinion on this is, this is still one-hot encoding + weight learning. However, we are forcing certain structures onto the weight matrix.

Feature selection 

Still based on one-hot encoding. However, instead of compressing everything into a tiny low-d vector, we discard some dummy variables based on their importance. In fact, LASSO is exactly used for this! L1 usually drives the coefficient of some features to zero, due to the diamond shape of the constraint. Source:

On why l1 gives sparsity: video here :  https://www.youtube.com/watch?v=14MKVkhvMus&feature=youtu.be
Course here : https://onlinecourses.science.psu.edu/stat857/book/export/html/137 Statoverflow answer here: https://stats.stackexchange.com/questions/74542/why-does-the-lasso-provide-variable-selection
Domain specific methods

These models exploit the relationship between these symbols in the training set, and learn a vector representation of every symbol (e.g. word2vec). This is certainly another way of vectorizing the words. Probably I will write more about this after learning more on representation learning!

