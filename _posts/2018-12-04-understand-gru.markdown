---
layout: post
title:  "Understanding GRUs"
date:   2018-12-04 20:38:34 -0400
categories: jekyll update
---

(This is a restoration of a previous post hosted on Wordpress. Hyperlinks might be missing and formatting might be a bit messy.)

This post follows this post last year about vanilla Recurrent Neural Network structures.

One of the ultimate goal of a recurrent neural network is to summarize a sequence of vectors into a single vector representing all the information.

In the simplest case, imagine for the past month, every day  you and your friend did some push-ups. Let's denote your number as x_i at day i, and your friend's number as y_i at day i. Now a month has passed and you two want to compare who is more keen on working out.

So what you do you? Directly comparing two vectors is hard, because there is no obvious "greater-than" relations across dimensions? One way is to calculate the simple average of the two sequences, i.e. on average you did \bar{x} and your friend did \bar{y} per day. Whichever number is larger, you can bet that person is a better athlete!

Now in this example, time order does not really matter, because we are treating numbers from 30 days ago as the same as number yesterday. Another example where time does matter is interests from bank savings. Suppose every year you save b_i amount of Yuan into the bank, and the bank pays out interests at the end of year, once per year. How much will your total savings be at the end of the fifth year? You can image the oldest money will receive the highest interests, because of confounding.

With the same logic, we can apply this idea to summarizing sentences. Suppose every word can be represented as a n-D vector in a semantic space. How do we produce one single vector that represent the meaning of this sentence?

Taking average is a way, but note every word will have some influence on the words after it. e.g. "this burger is delicious." Note how the "burger" constrains word choices after "is" ... And that's why we need recurrent neural network : at every step of the model, the hidden vector (which contains information about previous words) will concatenate with the current word, and becomes the input into producing the next hidden vector. So every word will have a lasting influence in the final output.

However, simple RNN has a vanishing (explosive) gradient problem. Mathematically, the older a word is, the higher order it will has on the multiplication factor of weight matrix. When taking first-order gradients, the weight matrix will have an order of n-1. Now if one term is larger than 1, as n approaches infinity this will will approach infinity too. If one term is smaller than 1, as n approaches infinity this will will approach zero and thus the model will "stop learning" (i.e. weights will not update).

Let's formulate the intuition above more formally. For a simple RNN, every update we have

h_t =g (W x_t + U h_{t-1} + b) 

Here g can be any non-linear activation function, e.g. a RELU or a sigmoid. Now we consider the gradient of h_t with regard to W.

Write

h_t = g (O_t)

and

O_t =W x_t + U h_{t-1} + b

Using the chain rule we have:

\frac{\partial h_t}{\partial W} =\frac{\partial h_t}{\partial O_t} \frac{\partial O_t}{\partial W}

We also know:

\frac{\partial O_t}{\partial W} = X_t + U_h \frac{\partial h_{t-1}}{\partial W}

So plug in the second equation above into the first equation, we have:

\frac{\partial h_t}{\partial W} = {\partial g} \cdot (X_t + U_h \frac{\partial h_{t-1}}{\partial W})

We can already see a recurrent relation between \frac{\partial h_t}{\partial W} and \frac{\partial h_{t-1}}{\partial W}. To make things clear, write \frac{\partial h_t}{\partial W} = \alpha_t, and expand the equation a few more steps:

\alpha_t = {\partial g} \cdot (X_t + U_h \alpha_{t-1})

\alpha_t = {\partial g} \cdot (X_t + U_h \cdot \partial g \cdot (X_{t-1} + \alpha_{t-2}) )

\ldots

\alpha_t = C + (\partial g U_h)^{n-1} \alpha_0

Here $C$ represent the other terms (with lower order of \alpha_t-s) in the formula that we omitted for simplicity. So we can see, as n increases, if mode of \partial g U_h is greater than 1 this term will approach infinity, or if is less than 1 it will approach zero.

The main reason is the same term is multiplied n-1 times. i.e. information always flow at the same rate every step. If you think about a sentence, this does not really make sense as words meaning / importance does not really decrease (increase) exponentially w.r.t. it's distance to the end of the sentence. The first word might very well be very important (e.g. a female name which will influence later pronoun choices "he" or "her"), but this vanilla RNN structure is too simple to capture that. We need more dynamic structures to allow information flow freely between time stamps.

Gated recurrent unit solves the problem by adapting different terms in every update step. There are two key ideas:

Introduces an external "memory" vector to capture long distance dependencies.
Allow error messages to flow at different strengths depending on inputs.
It achieves this by first computing two "gates":

update gate : z_t = \sigma (W_z x_t + U_z h_{t-1} + b_z)

reset gate: r_t = \sigma (W_r x_t + U_r h_{t-1} + b_r)

They are both continuous vectors of the same length as the hidden state, constructed by passing the current word x_t and the last hidden vector h_{t-1} through an MLP. The sigmoid function makes every element between 0 and 1, so when used to perform element-wise multiplications \o, these two gates essentially controls how much information "flows" through.

There is also a candidate vector representing "new memory content":

\tilde{h_t} = \text{tanh} (W_h X_t + r_t \circ (U_h h_{t-1} + b_h)

The usage of tanh and W_h X_t is pretty standard as in all other unites. But note the meaning of reset gate r_t here. If r_t is close to zero, then we "forget" all the previous information in the h_{t-1} vector, and just use the current word information.

An example where the reset gate is useful is sentiment analysis on a movie review, where the author spend many sentences describing and summarizing the movie plot, but conclude that "But the movie is really boring". With this sentence, all the previous information become useless. This reset gate can help the model "forget" the previous summaries.

Finally, we update the new hidden vector as :

h_t =  z_t \circ h_{t-1} + (1 - z_t) \circ \tilde{h_t}

You can see the update controls how much information is remembered from old state, and how much information is acquired from the new memory content.

A caveat is why do we need the reset gate at all now that we have the update gate? Doesn't we already have a gate to control how much old information is retained by using an update gate? Doesn't the update gate also have the ability to eliminate all historical information embedded in h_{t-1}? I did not find satisfactory information about this point online.

Now think about why GRU can solve the vanish gradient problem. The update gate allows us to retain all previous information by setting all elements of z_t to 1, and h_t is exactly the same as h_{t-1}. In this case, we do not have the vanishing gradient problem because no weight matrix is multiplied.

For example, in the same movie review suppose the author says "I love this movie so much" and then goes on to explain the movie plot. Now the first sentence is really important, but with recurrent neural network, we update the gradient at every step and the content will be washed out as time passes. But now the update gate allows the model to retain the first sentence without exponentially decay its content.

Units with short term memories usually have reset gate very activate (i.e. numbers close to 1), and thus forget most about the past...

If reset gate is entirely 1 and update gate is entirely zero, then we just have a standard RNN.

Why tanh and sigmoid?

In theory tanh can also be RElu, but in practice it just performs better.
Sigmoid will control value between 0 and 1, but again no formal justification.
Finally, here is an illustration that pictures what's going on in a GRU unit:



Reference:

This course video by Stanford.
This reddit discussion about whether the reset gate and update is really necessary.
This paper that explores thousands variants of LSTM. It turns out this reset and update structure is very stable and thus it is kept like this...
