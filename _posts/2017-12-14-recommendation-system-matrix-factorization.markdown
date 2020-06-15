---
layout: post
title:  "Recommendation System: Matrix Factorization"
date:   2017-12-14 17:13:34 -0400
categories: jekyll update
---

(This is a restoration of a previous post hosted on Wordpress. Hyperlinks might be missing and formatting might be a bit messy.)

This post explains how to build a recommendation system based on matrix factorization. However in practice, the engineering aspects are more challenging when the dataset is huge - these are not covered here.

Recommendation system:

The goal is to recommend an item to a user based on this user's previous ratings on the item. Such systems are now used everywhere.

The formulation of this problem is to build a utility matrix, in which:

Each row represent a user
Each column represent an item
The elements in that matrix can be 1) user rating on this item or 2) frequency user visited this item (e.g. in the context of Spotify)
The problem is such matrix is usually sparse. Not every user has rated every item. Thus the goal is to fill in the blanks.

After filling in the blanks, we can recommend the items with high ratings but the user has not yet reviewed, to the user.

Algorithms

Two sets of algorithms:

Collaborative filtering --- use clustering to find similar rows/columns, then compute mean
Matrix Factorization  --- as the name indicates.
We only talk about matrix factorization here.

The utility matrix can be decomposed as :

U = \Sigma \cdot V^{T}

If U is a m * n matrix, Now $\Sigma$ is a m * k matrix, and V is a n * k matrix.

We can interpret U as :

each column represent a latent factor
each row represent a user
each element represents the users' preference to that latent factor
We can interpret V as :

each column represent a latent factor
each row represent an item
each element represents the latent factor's contribution to that item in rating
This looks complex but in fact is really simple. We basically interpret the rating of user i on item j, as the summarization of user i's ratings on each latent factor!

An example:

I rate the book Little Prince a 3.6/5.
I rate, because Little Prince is 20% about youth and 80% about romance.
I give the youth factor 2 score.
I give the romance factor 4 score.
My total score of little prince is then 2 * 0.2 + 4 * 0.8 = 3.6.
Compute matrix factorization

The above is about a theoretical, fully filled utility matrix.

Now our goal is to find such a matrix, by finding corresponding \Sigma and V.

We can view this as a supervised learning problem.

The observed value is the observed user-item utility matrix.

The theoretical value is $\Sigma \cdot V$.

The cost function is square loss, for every i and j in that matrix.

Our goal is to minimize the empirical cost function - the sum of all square loss. We can of course add regularization terms to this.

How to achieve this? Using iteration.

First randomly initialize  \Sigma andV.
Then choose one element (parameter) in \Sigma or V. Set this one to be the argmin of the empirical cost function
Change to another parameter and iterate the process, until convergence.
Now the following is to be noticed:

Proprocessing of matrix: since different users have different rating scale, can normalize the utilitity matrix by row, by column.
How to choose the order in choosing parameters?
Can randomly choose!
Can choose by column!
Overfit:
Early stop.
Till now I have described this algorithm. It turns out to be pretty simple! Took me 15 minutes to understand and 18 minutes to write this post. Next time when learning techniques, should aim for the same time span.

Reference:

The Stanford textbook : http://infolab.stanford.edu/~ullman/mmds/ch9.pdf

