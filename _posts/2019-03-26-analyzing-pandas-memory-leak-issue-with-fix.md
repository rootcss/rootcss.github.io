---
layout: post
title: Analyzing Python Pandas' memory leak and the fix
tags:
- Python
- Pandas
---

At <a target="_blank" href="https://getsimpl.com">Simpl</a>, we use <a target="_blank" href="https://pandas.pydata.org/">pandas</a> heavily to run a bunch of our machine learning models many of them implemented with scikit-learn. We've been growing rapidly and sometime back, one of the models crashed with python's `MemoryError` exception. We were pretty sure that the hardware resources are enough to run the task.

What is the <b>MemoryError</b>? It's an exception thrown by interpreter when not enough memory is available for creation of new python objects or for a running operation. 

The catch here is that, it doesn't necessarily mean "not enough memory available". It could also mean that, there are some objects that are still not cleaned up by Garbage Cleaner (GC).

To test this, I wrote a very small script:
```python
arr = numpy.random.randn(10000000, 5)

def blast():
    for i in range(10000):
        x = pandas.DataFrame(arr.copy())
        result = x.xs(1000)

blast()
```

Below is the distribution (Memory usage w.r.t Time) <u>before the program crashed</u> with MemoryError exception.

<p><img class="img-responsive" src="{{ site.url }}/assets/images/pandas_memory_leak.png" alt="Pandas memory leak" /></p>

The GC seems to be working fine, but it's not able to clean up the objects as fast as it's required in this case.
<br><br>

<b>What's the issue?</b><br>
Python's default implementation is `CPython` (<a target="_blank" href="https://github.com/python/cpython">github</a>) which is implemented in C. The problem was <a target="_blank" href="https://sourceware.org/bugzilla/show_bug.cgi?id=14827">this</a> bug; in the implementation of `malloc` in `glibc` (which is GNU's implementation of C standard library).
<br><br>

<b>Issue Details</b>:<br>
`M_MXFAST` is the maximum size of a requested block that is served by using optimized memory containers called `fastbins`. `free()` is called when a memory cleanup of allocated space is required; which triggers the trimming of fastbins. Apparently, when `malloc()` is less than `M_MXFAST`, `free()` is not trimming fastbins. But, if we manually call `malloc_trim(0)` at that point, it should free() up those fastbins as well.

Here is a snippet from `malloc.c`'s `free()` implementation (alias `__libc_free`). (<a target="_blank" href="https://code.woboq.org/userspace/glibc/malloc/malloc.c.html#3115">link</a>)

```c
  p = mem2chunk (mem);
  if (chunk_is_mmapped (p))                       /* release mmapped memory. */
    {
      /* See if the dynamic brk/mmap threshold needs adjusting.
         Dumped fake mmapped chunks do not affect the threshold.  */
      if (!mp_.no_dyn_threshold
          && chunksize_nomask (p) > mp_.mmap_threshold
          && chunksize_nomask (p) <= DEFAULT_MMAP_THRESHOLD_MAX
          && !DUMPED_MAIN_ARENA_CHUNK (p))
        {
          mp_.mmap_threshold = chunksize (p);
          mp_.trim_threshold = 2 * mp_.mmap_threshold;
          LIBC_PROBE (memory_mallopt_free_dyn_thresholds, 2,
                      mp_.mmap_threshold, mp_.trim_threshold);
        }
      munmap_chunk (p);
      return;
    }
```

Therefore, we need to trigger `malloc_trim(0)` from our python code written above; which we can easily do using `ctypes` module.
<br><br>
The fixed implementation looks like this:

```python
from ctypes import cdll, CDLL
cdll.LoadLibrary("libc.so.6")
libc = CDLL("libc.so.6")
libc.malloc_trim(0)

arr = numpy.random.randn(10000000, 5)

def blast():
    for i in range(10000):
        x = pandas.DataFrame(arr.copy())
        result = x.xs(1000)
        libc.malloc_trim(0)

blast()
```
<br>
In another solution, I tried forcing the GC using python's `gc` module; which gave the results similar to above method.

```python
import gc

arr = numpy.random.randn(10000000, 5)

def blast():
    for i in range(10000):
        x = pandas.DataFrame(arr.copy())
        result = x.xs(1000)
        gc.collect() # Forced GC

blast()
```
<br>
The distrubution of Memory usage w.r.t Time looked much better now, and there was almost no difference in execution time. (see the "dark blue" line)
<p><img class="img-responsive" src="{{ site.url }}/assets/images/pandas_memory_leak_fix.png" alt="Pandas memory leak with fix" /></p>

<br><br>
Similar cases, References and other notes:
1. Even after doing `low_memory=False` while reading a CSV using `pandas.read_csv`, it crashes with `MemoryError` exception, even though the CSV is not bigger than the RAM.
2. Explanation of malloc(), calloc(), free(), realloc() deserves a separate post altogether. I'll post that soon.
3. Similar reported issues:<br>
        - https://github.com/pandas-dev/pandas/issues/2659<br>
        - https://github.com/pandas-dev/pandas/issues/21353<br>

