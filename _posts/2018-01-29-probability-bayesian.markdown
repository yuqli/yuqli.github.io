---
layout: post
title:  "Probability, statistics, frequentist and Bayesian"
date:   2018-01-29 10:55:34 -0400
categories: jekyll update
---
(This is a restoration of a previous post hosted on Wordpress. Hyperlinks might be missing and formatting might be a bit messy.)

This post is a review of basic concepts in probability and statistics.

Useful reference: https://cims.nyu.edu/~cfgranda/pages/DSGA1002_fall15/notes.html

https://ocw.mit.edu/courses/mathematics/18-05-introduction-to-probability-and-statistics-spring-2014/

Probability

It's a tool to mathematically measure uncertainty.

Formal definition involving \sigma-algebra:

A probability space isa triple (\Omega, F, P) consisting of :

A sample space \Omega
A set of events F - which will be \sigma-algebra
A probability measure P that assigns probabbilites to the events in F.
Example: We have a fair coin. Now we toss it 1000 times, what's the probability of getting 600 heads or more?

Statistics

The goal of statistics is to 1) draw conclusion from data (e.g. reject Null Hypothesis) and 2) evaluate the uncertainty of this information (e.g. p-value, confidence interval, or posterier distribution).

At the bottem, statistical statement is also about probability. Because it applies probability to draw conclusions from data.

Example: We would like to know whether the probability of raining tomorrow is 0.99. Then tomorrow comes, and it does not rain. Do we conclude that P(rain) = 0.99 is true?

Example 2: We would like to decide if a coin is fair. (Data) Toss the coin 1000 times, and 809 times it's a head. Do we conclude the coin is fair?

Note : probability is logically self-contained. There are a few rules, and the answers follow from the rules. Statistics can be messy, because it involves draw conclusion from data - much art than science.

Frequentist vs Bayesian

Two schools of statistics. They are different in their interpretation of probability.

Frequentist interpret probability to be the frequencies of events in repeating experiments. E.g. P(head) = 0.6. Then if we toss a coin 1000 times, we will have 600 heads.

Bayesian interprets probability to be a state of knowledge, or a state of belief, about a preposition. E.g. P(head) = 0.6, means we are fairly certain (around 60% certain!) that a coin will be tossed head.

In practice though, Bayesian seldom use a single value to characterize such belief. Rather, it uses a distribution.

Frequentists are used in social science, biology, medicine, public health. We see two sample t-tests, p-values. Bayesian is used in computer science, "big data".

Core difference between Frequentists and Bayesian

Bayesian considers the results from previous experiments, in the form of a prior.

See this comic for an illustration.

What does it mean?

A frequentist and a Bayesian are making a bet about whether the sun has exploded.

It's night, so they can not observe.

They ask some expert whether the sun has gone Nova.

They also know that this expert will toss two coins. If both get 6, she will lie. Else, she won't. (Data generation process)

Now they ask the expert, who tells them yes, the sun has gone Nova.

Frequent conclude that since the probability of getting two 6's is 1/36 = 0.0027 <0.05 (p < 0.05), it's very unlikely the expert has lied. Thus, she concludes the expert did not lie. Thus, she concludes that the sun has exploded.

Bayesian, however, has a strong belief that the sun has not exploded (or else they will be dead already). The prior distribution is

P(sun has not exploded) = 0.99999999999999999,
P(sun has exploded) = 0.00000000000000001.
Now the data generation process is essentially the following distribution:

P(expert says sun exploded |Sun not exploded) =  1/36.
P(expert says sun exploded |Sun exploded) =  35/36.
P(expert says sun not exploded |Sun exploded) =  1/36.
P(expert says sun not exploded |Sun not exploded) =  35/36.
The observed data is "expert says sun exploded". We want to know

P( Sun exploded | expert says sun exploded ) = P( expert says sun exploded | Sun exploded) * P( Sun exploded) / P(expert says sun exploded)
Since P(Sun exploded) is extremely small compared to other probabilities, P( Sun exploded | expert says sun exploded ) is also extremely small.

Thus although the expert is unlikely to lie (p = 0.0027), the sun is much more unlikely to have exploded. Thus, the expert most likely lied, and the sun has not exploded.

