---
layout: lesson
---

Writing code in python is easy: because it is dynamically typed, we don't
have to worry to much about declaring variable types (e.g. integers vs.
floating point numbers). Also, it is interpreted, rather than compiled.
Taken together, this means that we can avoid a lot of the boiler-plate
that makes compiled, statically typed languages hard to read. However,
this incurs a major drawback: performance for some operations can be
quite slow.

Whenever possible, the numpy array representation is helpful in saving
time. But not all operations can be vectorized.

Here, we introduce three new technologies that can substantially speed up
computations in Python: Cython, Numba and Dask.

Cython takes code that is written in python, and, provided some
additional amount of (mostly type) information, compiles it to C, then
compiles the C code, and bundle the C objects into python extensions that
can then be imported directly into python. Materials on Cython are based
partially on a [tutorial](https://www.youtube.com/watch?v=JKCjsRDffXo)
given by Kurt Smith (Enthought) at the Scipy conference, 2013.

Numba takes a different approach, using just-in-time compilation. Numba
is surprisingly simple to use, and the benefits can be staggering.

Dask parallelizes computations across different threads, the different
cores that are available to us on the machine we are using, or across a
cluster.

The template we use for these lesson websites is based on the lesson
template used in [Data Carpentry]({{ site.dc_site }}) and [Software
Carpentry]({{site.swc_site }}) workshops,