---
layout: post
title:  "Distributed Stochastic Gradient Descent with MPI and PyTorch"
date:   2018-12-16 21:12:34 -0400
categories: jekyll update
---
(This is a restoration of a previous post hosted on Wordpress. Hyperlinks might be missing and formatting might be a bit messy.)

This post describes how to implement stochastic gradient in a distributed fashion with MPI. It will cover the following topics in a high-level fashion, as it is challenging to cover every details in a single post. I will point to other resources helpful in understanding key concepts.

What is MPI
How to do distributed SGD
PyTorch modules necessary for writing distributed SGD and how to design the program
Engineering caveats for building PyTorch with MPI backend 
What is MPI

I understand MPI as a programming framework that handles communication between computers. 

It's like the lower-level software underneath MapReduce, another programming framework for distributed computing. In MapReduce you need to specify a map operation and a reduce operation and that's all. The system take care of the allocation of workers and server and memory, which can of course taken care of by meta-information input into a shell script. The main challenge is designing the parallel algorithms at the algorithmic level, for example how to parse the two matrices when doing matrix multiplication, or how to design the mapper and reducer for min-hash and shingling documents. 

But in MPI, the programmer needs to specify the worker and server by their ID, and actively coordinate the computers, which probably requires another set of algorithms. For example, if many workers need to communicate with the server at the same time but the server cannot serve them all, there needs to be a scheduling algorithm e.g. round-robin. In MPI, the programmer needs to code the scheduling by themselves so that messages from different workers won't be mixed. We'll see an example later in the asynchronous SGD. 

MPI has many concrete implementations, for example OpenMPI and MPICH. 

MPI has the following basic operations:

point-to-point communications: blocking and non-blocking. This enables a single computer to send message to another, and another computer to receive message. blocking means the process is blocked until the send / receive operation is finished, while block means the process is returned immediately, not wait till the process is finished.
collective communications : examples are reduce and gather.
Here is a good place to learn basic operations of MPI. You need to go through all tutorials on that website to be able to understand and write a project that does distributed SGD with PyTorch.

How to do distributed SGD

There are two types of distributed SGD, depending on when to synchronize gradients computed on individual workers.

 synchronous SGD:
all workers have a copy of the model. they are all the same
at every mini-batch, all workers compute their share of gradient, and then compute average
the model update on every worker
then move on to the next batch
if there is a server,
the server has a copy of the model
the workers also have copies of the model
workers are assigned data to calculate forward and backward
gradients are sent to the server to take care of the averaging
workers receive updated gradients from the server
if there is no server, the gradient is calculated in all_reduce. all_reduce can (but not necessarily) be implemented in the ring algorithm: every model send the results to its neighbor ?
asynchronous SGD:
all workers have a copy of the model, but they are not the same. the model is inconsistent across workers
at every mini-batch:
workers get parameters from the server
workers get data from its own data loader, or randomly selected dataset (is there an epoch concept anymore?)
workers calculate forward and gradient
once the calculation is done, gradient is sent to the server,
the server updates the parameters
Now we need to conceptually understand how this workflow can be implemented using data structure and APIs provided by PyTorch. To do this, let's first analyze how a single machine SGD is implemented in PyTorch.

In particular, in a single machine SGD implemented with PyTorch, 1) the output of a mini-batch is calculated by calling the forward function of the model, 2) the loss is calculated by sending the output and the target to a loss function, and 3) the gradient is calculated by calling loss.backward, and 4) the update to parameters is done by calling optimizer.step(), and we would pass the model parameters to the optimizer beforehand.

Now for this single-machine workflow, key functions are:

We calculate the gradients in the loss.backward() step
The gradients are stored in model.parameters()
The model parameters are updated when we call optimizer.step()
So, in a multi-machine scenario, the following questions need to be considered:

How many copies of the model should be created?
What should the server store?
What should the slaves store?
What should they communicate with each other?
To see why 1) is important, note the way we deploy MPI is by having the same script sending to every machine, but use MPI_rank to identify servers and slaves, and use if-else condition to decide which block of code should be run on which machine. So, we can theoretically create some objects when we encounter a server, but do not create these objects when we encounter a slave. To answer 1), an obvious answer is everybody has its own copy of a whole model (i.e. the whole computation graph created by model = myModelClass()), but is this necessary?

It turned out it is. Why? Although server only need the parameter value and gradient values to perform updates, and theoretically we only need to put the optimizer on server, and model parameters on slaves, this is not doable in operation because in PyTorch optimizer is tied to a set of model parameters. More, the whole computation graph contains more than data, but also relations. So, we must initialize a model and a optimizer on both server and slaves, and use communication to make sure their values are on the same page.

To answer question 4, here is the logic flow:

a slave establishes connection with the server
a slave fetches initial parameter values from the server
a slave breaks connection with the server
a slave calculate the loss on its own batch of data
a slave calculates the gradients by calling loss.backward()
a slave establishes connection with the server
a slave send the gradients and loss to the server
a slave breaks connection with the server
the server updates its parameter value.
Note here we have multiple steps concerning how to set up connections. In effect, parameter values have many layers corresponding to different layers of a neural network, and we have multiple salves. So if multiple salves trying to send the same set of parameters to the server, data from different sets might be messed up. In other words, MPI needs to know who sends data and programmers need to use a protocol to ensure that during the transmission, only one pair of connection is in effect.

Note MPI does not allow one slave to "block" other slaves, so we need to code this "block" up by using "handshake" technique in TCP. The idea is a slave first send a "hello" message to the server, and when a server is busy this message will wait in a queue until being received. And when the server is idle, and it receives the "hello" message, it will realize someone else is calling it and will wait for this particular person by only receiving message from the same ID for this round. It will also send a message to this ID, telling it "I'm ready for you and no one else is able to mess around". When a slave receives this confirmation from the server, it can go on to send the important gradient information to the server. After it finishes, it will receive the information send from the server. And when the server finished sending update information to the slave, it will be idle again and able to receive "hello" from another slave. This is a very interesting process to establish connections!

Engineering caveats for building PyTorch with MPI backend 

Some caveats to watch out for:

To use PyTorch with MPI, it cannot be installed from Conda. You have to build it yourself. But PyTorch is a large package so check your disk space and act accordingly. It took me 4 days to build it successfully....
Some links:

PyTorch tutorial for distributed SGD. Note this guy uses a synchronized SGD so the process is easier - you can just run an all-reduce and you do not have to worry about slaves mess up with each other. https://pytorch.org/tutorials/intermediate/dist_tuto.html#advanced-topics
My implementation of the distributed SGD process... https://github.com/yuqli/hpc/blob/master/lab4/src/lab4_multiplenode.py
Finally, this blog post is not very clearly written. If someone really reads this and has questions, please email me at greenstone1564@gmail.com.