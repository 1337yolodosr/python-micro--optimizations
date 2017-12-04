# python-micro--optimizations
a handful of optimizations, tricks and cool things i've learned while doing python


# return statements in single else clauses at the end of functions


```py
def f(x):
  if x:
    return x
  else:
    return f(x+1)
```

Consider the following case; the `else` syntax is redundant as the prior `return` already breaks the flow of the program therefore there is no point for the redundant `else` clause.

```py
def f(x):
  if x:
    return x
  return f(x+1)
```


# when you should be using `if not x`, `if x`, `if x == 1`, `if x == 0`, `if x is None` or `if x == None`

`if not x`: evaluates to True for when:
              `x` -> `0`
The condition does not evaluate to True for negatives or any positives -- nor any special float values like `inf` or `nan`
 
`if x`: evaluates to True for when:
              `1` <= `x` <= `inf`
              `-inf` <= `x` < `0`
The condition does not evaluate to True just for `1`, so keep in mind when doing `if x` (where `x` is arbitrary) that you're not assuming `x` to be `1` (unless within logical reason) and treating it as so.

`if x == 1`: evaluates to True for when:
              `x` -> `1`
When doing any form of code-review, don't note something like this as a request for simplification to something like `if x` - since as explained above - unless the conditions exempt this malpractice, you cannot always assume that `x` is `1`, but rather between `-inf, -1 and 1, inf`.

`if x == 0`: evaluates to True for when:
              `x` -> `0`
Unlike previous explanations, this can be utmostly simplified to `if not x`.

`if x is None`: evaluates to True for when:
              `x` -> `NoneType`
This is the preferred form of checking whether arbitrary `x` is `None` as it is faster (since it doesn't call an underyling `__eq__`).

`if x == None`: evaluates to True for when:
              `x` -> `NoneType`
Unlike the relative former example (`if x is None`), this calls the underlying (yet seemingly unexistent/unimplemented-- clarify?) `__eq__` function, which provides call overhead and thus slows it down.

```
 unazed@unazed  ~  python3.7 -m timeit --setup "a = None" -c "a is None" 
10000000 loops, best of 5: 27.9 nsec per loop
 unazed@unazed  ~  python3.7 -m timeit --setup "a = None" -c "a == None" 
10000000 loops, best of 5: 35.9 nsec per loop

 unazed@unazed  ~  python3.6 -m timeit --setup "a = None" -c "a is None"
10000000 loops, best of 3: 0.0245 usec per loop
 unazed@unazed  ~  python3.6 -m timeit --setup "a = None" -c "a == None"
10000000 loops, best of 3: 0.0352 usec per loop

 unazed@unazed  ~  python3.2 -m timeit --setup "a = None" -c "a is None" 
10000000 loops, best of 3: 0.0283 usec per loop               
 unazed@unazed  ~  python3.2 -m timeit --setup "a = None" -c "a == None" 
10000000 loops, best of 3: 0.0362 usec per loop

 unazed@unazed  ~  python2.7 -m timeit --setup "a = None" -c "a is None" 
10000000 loops, best of 3: 0.0326 usec per loop
 unazed@unazed  ~  python2.7 -m timeit --setup "a = None" -c "a == None" 
10000000 loops, best of 3: 0.0547 usec per loop

 unazed@unazed  ~  python2.0
...
>>> start = time.time(); good(None); print("Took: %f seconds" % (time.time()-start))
# GOOD
Took: 0.001744 seconds
>>> start = time.time(); bad(None); print("Took: %f seconds" % (time.time()-start))
# BAD
Took: 0.001774 seconds

 unazed@unazed  ~  python1.6
... 
>>> start = time.time(); good(None); print("Took: %f" % (time.time()-start))
Took: 0.001356
>>> start = time.time(); bad(None); print("Took: %f" % (time.time()-start))
Took: 0.001371

```

Neither Python 3.7.0a2, 3.6.3, 3.2.3, 2.7.14, 2.0.1, 1.6.1 showed any difference for where the `x == None` time was less than `x is None`.