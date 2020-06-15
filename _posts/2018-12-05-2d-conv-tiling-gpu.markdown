---
layout: post
title:  "2D convolution with tiling technique on GPU"
date:   2018-12-05 22:42:34 -0400
categories: jekyll update
---
(This is a restoration of a previous post hosted on Wordpress. Hyperlinks might be missing and formatting might be a bit messy.)

This post describes how to write CUDA C code to perform 2D convolution on GPU with tiling technique. It is very brief, only covers basic concepts but with links to code examples. We are not going to use cuDNN, only the bare bones of CUDA. But here is an excellent post if you want to use cuDNN - for this particular task, cuDNN is slower than the code I will present below.

You can find the source code for this post here.

This post will consist of the following sections:

Installation CUDA
Basic GPU programming model
Vanilla convolution on GPU
Constant memory in GPU
Tiling technique and indexing
Link to code example
Install CUDA

First, make sure if you have a NVIDIA GPU on your machine. Here is how. Or just search the model online and ask on reddit 🙂

Next, follow the official NVIDIA guide here to download CUDA Toolkit. Note not every card support every version of CUDA kit.

Note also the download path, as you will need to specify that to the compiler later. If you are using Linux you can type whereis nvcc to find out. nvcc is the NVIDIA C compiler.

It's easy to get wrong. Just follow carefully every step in the official installation guide. In particular don't forget to finish the post-installation set-ups! Wrong PATH environment variables can be a real pain.

Basic CUDA Programming model

The most prominent characteristic of CUDA programming is parallization. When you write one line of code in a CUDA kernel (explained later), it will be executed by thousands of threads on different data. This is called the SIMD (same instruction multiple data) parallel paradigm.

With this idea in mind, here is what a typical CUDA C code look like:

Setting up the program on the host (CPU) by initializing variables and populate them. Write non-parallel sequential code.
Preparing for GPU work by initializing device (GPU) variables that reside in device memory, and copy contents from the corresponding host variables to it.
Write a GPU kernel function to deal with the parallel part of computation (e.g. convolution).
Set up the parallel configurations by specifying how many blocks and grids are used for the kernel. These two variables will decide how many threads we started for parallel execution.
Call the kernel from host code. Store the results in a device variable.
Copy contents from device to the host.
Some concepts to clear:

How many threads we use in the GPU is controlled by Grid size and Block size. A grid consists of many blocks. Grid and Block are both 3D structures. For example, a grid of dimension <2, 2, 2> will have 8 blocks in total, and a block of dimension <2, 2, 2> will have 8 threads in total. A block cannot have more than 1024 threads.
Here is a basic matrix addition example:

 

Vanilla convolution on GPU

Where to parallelize for convolution? We use every thread to correspond to one single element in the output, and all threads do the same convolution operation but with different data chunks.

Constant memory in GPU

Constant memory variables are shared by all threads in all blocks. This memory is small but fast. So we should put the filter in it because everyone needs it, and it's small.

Tiling technique and indexing

Tiling is making use of the shared memory on GPU. Shared memory is shared by all threads in the same block, it is small but fast. So the idea is for every thread in a block, they will operate on data regions that have some overlaps. And we make use of that to copy the small region of the larger picture that these threads need into the shared memory and dealing with them.

We use all threads to copy these data elements. Here is a further illustration of this tiling idea.

Code example

Here.

 

Here is the tiling organization :


