---
title: "Introduction"
teaching: 10
exercises: 5
questions:
- "Why is Python slow?"
- "Why is Python fast?"
- "What are some ways you can speed Python up even more?"
objectives:
- "Profile functions with ipython magic functions"
key points:
- "Python is slow"
- "Python is fast when you can vectorize it"
- "There are several approaches to speeding up code beyond vectorizing"
- "Profiling code is a useful way to know whether some change is helping"
---

### Introduction

Writing code in python is easy: because it is dynamically typed, we don't have
to worry to much about declaring variable types (e.g. integers vs. floating
point numbers). Also, it is interpreted, rather than compiled. Taken together,
this means that we can avoid a lot of the boiler-plate that makes compiled,
statically typed languages hard to read. However, this incurs a major drawback:
performance for some operations can be quite slow.

Whenever possible, the numpy array representation is helpful in saving
time. But not all operations can be vectorized. What do you do when you need
to speed up your code, but can't rely on vectorization?

Here, we'll explore three approaches to speeding up code:

1. Sometimes, your only choice in speeding up code is to write extension
   code in C, but this is very cumbersome, and requires writing many
   lines of additional code above and beyond your core algorithms, just
   to communicate between the Python and C computation layers.
   [Cython](http://cython.org/) is a technology that allows us to easily
   bridge between python, and the underlying C representations. The main
   purpose of the library is to take code that is written in python, and,
   provided some additional amount of (mostly type) information, compile
   it to C, compile the C code, and bundle the C objects into python
   extensions that can then be imported directly into python.

2. [Numba](https://numba.pydata.org/) also compiles your code to machine
   code, but it takes a distinctly different approach. Instead of
   translating your Python code to C, and then compiling that down to, it
   relies on the [LLVM compiler infrastructure](http://llvm.org/), to
   compile the code "just in time", at the time that the code is called.

3. Another approach to speeding up execution of code is by parallelizing
   it. Most of the time (but not always), Python code that you write will
   run on a single thread at any given time (because of the so-called
   Global Interpreter Lock, or GIL).

### Profiling

To know whether what you are doing is helping, it is crucial to measure
how well you are doing before and after some change. Profiling is a way
to know how well a particular piece of code works.

#### The IPython `timeit` magic

In the Jupyter Python notebook, you can use a 'magic' function to time
either a single statement, or multiple statements.

For example:

~~~
import numpy as np

for shape in [10e3, 10e4, 10e5]:
    X = np.random.rand(int(shape))
    %timeit np.dot(X, X.T)
~~~
{: .python}

Demonstrates how to time the scaling of one operation over inputs of
different sizes. The `%timeit` magic only times the operation on _that line_.

It should look something like this:

~~~
3.29 µs ± 31 ns per loop (mean ± std. dev. of 7 runs, 100000 loops each)
26.7 µs ± 364 ns per loop (mean ± std. dev. of 7 runs, 10000 loops each)
388 µs ± 15.4 µs per loop (mean ± std. dev. of 7 runs, 1000 loops each)
~~~
{: .python}

In contrast, if you use `%%timeit`, the magic would apply to the entire cell.

For example, in the following cell, we might calculate the pair-wise
distance between the entries in a random matrix of 100 by 100, and store
them:

~~~
X = np.random.rand(100, 100)
D = np.empty((100, 100))

M = X.shape[0]
N = X.shape[1]
for i in range(M):
    for j in range(M):
        d = 0.0
        for k in range(N):
            tmp = X[i, k] - X[j, k]
            d += tmp * tmp
        D[i, j] = np.sqrt(d)
~~~
{: .python}


This might take something like:

~~~
468 ms ± 3.38 ms per loop (mean ± std. dev. of 7 runs, 1 loop each)
~~~

### Line profiling

Knowing that some set of procedures takes time is good, but to improve
things, often need to drill down deeper, and figure out which exact lines
within a function are the ones that take up most of the time.

That's where a line-profiler comes in handy. To install with conda:

~~~
conda install line_profiler  
~~~
{: .bash}

source:https://github.com/rkern/line_profiler
To use the line-profiler in the notebook, you'll first need to load the
line_profiler extension:


~~~
%load_ext line_profiler
~~~


One you've done that, you'll need to define a function around the code
that you are interested in profiling:

~~~
def distance():
    X = np.random.rand(100, 100)
    D = np.empty((100, 100))

    M = X.shape[0]
    N = X.shape[1]
    for i in range(M):
        for j in range(M):
            d = 0.0
            for k in range(N):
                tmp = X[i, k] - X[j, k]
                d += tmp * tmp
            D[i, j] = np.sqrt(d)
~~~
{: .python}

Sometimes the function you want to profile is not the same as the one you
would call to profile it, so the syntax of the line-profiler extension
is:

~~~
%lprun -f function_to_be_profiled function_to_be_called()
~~~

In this case, they are the same, so running the following:

~~~
%lprun -f distance distance()
~~~

Would produce something like this:

~~~
Timer unit: 1e-06 s

Total time: 2.21674 s
File: <ipython-input-5-2dd6126db7de>
Function: distance at line 1

Line #      Hits         Time  Per Hit   % Time  Line Contents
==============================================================
     1                                           def distance():
     2         1        289.0    289.0      0.0      X = np.random.rand(100, 100)
     3         1         17.0     17.0      0.0      D = np.empty((100, 100))
     4
     5         1          3.0      3.0      0.0      M = X.shape[0]
     6         1          0.0      0.0      0.0      N = X.shape[1]
     7       101        107.0      1.1      0.0      for i in range(M):
     8     10100       5793.0      0.6      0.3          for j in range(M):
     9     10000       5209.0      0.5      0.2              d = 0.0
    10   1010000     522677.0      0.5     23.6              for k in range(N):
    11   1000000     945404.0      0.9     42.6                  tmp = X[i, k] - X[j, k]
    12   1000000     700715.0      0.7     31.6                  d += tmp * tmp
    13     10000      36526.0      3.7      1.6              D[i, j] = np.sqrt(d)
~~~
{: .python}


The 'Hits' column is important, because it tells us that some lines of
code are heavily used. And the '% Time' column is also very important,
because it tells us where we should focus our attention first, in making
this go faster.



> Similar to profiling execution time, sometimes you need to profile memory
> usage. There's a thing for that too!
> To install with conda:
>
>   `conda install memory_profiler`
>
> And to use:
>   `%load_ext memory_profiler`
>   `%mprun -f distance distance()`
>
> We will not demonstrate this here, but take a look at some examples
> in [Chapter 1 of Jake Vanderplas' book](https://jakevdp.github.io/PythonDataScienceHandbook/01.07-timing-and-profiling.html)
