---
layout: post
title:  "Writing Python tests: doctest, unittest, pytest"
date:   2018-11-24 20:23:34 -0400
categories: jekyll update
---
(This is a restoration of a previous post hosted on Wordpress. Hyperlinks might be missing and formatting might be a bit messy.)

So I set myself another task the other day: writing tests for an open-source project ---- a baby step towards software engineering! This post describes the tools and practices I've learnt.

The project I've committed to is here. The goal is to write tests for an experiment class, whose objective is to save state of machine learning experiments and quickly retrieve them.

There are two sets of tools to be learnt: 1) conceptually, what is a test and how to  write one? 2) practically, what tools does Python offer for writing and organizing test code, and how to use them? I will answer these two sets of questions below.

What is software testing and how to write it?

Testing is just write code to test if a software behaves as expected. If it does not, then there is a bug, and bugs are bad. Most of the online materials / books on this are kind of fluffy, describing concepts that are hard to define and unnecessarily convoluted. But really I just understand test to be like, this piece of software is supposed to output 1 when I input 3, is that what it is behaving?

There are also edge cases to be tested and taken care of accordingly.

So the real issue seems to be how to write good tests. This is a very broad topic with no definite answer, and I'll omit discussion in this post. For this project, I'm following this guide.

Testing tools in Python 

doctest
unittest
pytest
doctest  is packaged with Python. The usage is the following:

Open a Python interpreter, and interactively test a bunch of results with the function to be tested
Copy and paste all output from the interactive session to the docstring of that specific function
add a if __name__ == "__main__" line in the script to be tested so as to detect if it is imported or being executed standalone. If it is executed standalone, add one line doctest.testmod() so the doctest will run the test mode..
this module will parse the docstring to get expected results for every input. It will then compare the actual output with the expected input, and return errors if they do not match. If everything runs fine, the module will not output anything.

unittest the biggest difference with doctest is test cases are no longer defined in the original script. Instead, we now create a new script, import the old script and its methods, and test all out test cases there. The good news is now script is separated from tests, the bad news is we need to write more code.

How to use unittest module?

First create a new python script, import bothunittest and the original script in it. This script will be run alone, so need to specify the shebang line as well.
Create a unittest.TestCase class. syntax is class MyTestCase(unittest.TestCase), i.e. MyTestCase is a subclass of unittest.TestCase.
So the real issue is what methods we can use with unittest.TestCase. These methods are used to assert if the expected value is equal to the real reaults. Here is a link to the useful parts of its documentation. Basically, the testing process is a series of assertion processes.
The most naive way is just to define a bunch of assertTrue() methods in the class one by one. A more mature way is to use the setUp and tearDown methods to put all test cases into a data structure, e.g. a list, and in the test function body use a for loop to iterate through that data structure. Sample code here : https://www.python-course.eu/python3_tests.php
pytest is a third party package. It is easier to use than the unittest module, because it does not require all the class and method setups.

The simplest case, first create a python script exercise.py with 1) the function to be tested and a function to test that function with assert. e.g. assert func(4) == 5
Then, run the scrip with pytest exercise.py from command line in the folder containing the script. You do not even need to import pytest in exercise.py!
On the other hand, if we do not specify an argument to pytest, it will run test on every file of the form *_test.py or test_*.py in the current directory. This is a convention to discover test files. Also see this link.
Happy testing!