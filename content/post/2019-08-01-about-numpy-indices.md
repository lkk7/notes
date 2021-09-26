+++
title = "About numpy.indices"
date = 2018-08-05T00:00:00+02:00
description = "About numpy.indices"
tags = ["python"]
aplayer = false
showLicense = false
+++

`numpy.indices` was a function that I really couldn't "get" for a long time. But with help of [this Stack Overflow answer](http://stackoverflow.com/questions/32271331/can-anybody-explain-me-the-numpy-indices/32271420#32271420) (I am the question asker) I have grasped it. Here I will present the idea with examples.
<!--more-->
### The complicated explanation
We pass a `dimensions` argument made of `n` elements, for example `(7)` or `(2, 3)`. `numpy.indices(dimensions)` will return an array (a `numpy.ndarray`) made of `n` subarrays, in which each subarray is `dimensions`-shaped and contains its indices varying only in the corresponding dimension.
### The essential examples
The explanation may seem complicated, but the examples will make it clearer.

``` python
>>> import numpy as np
>>> result = np.indices([7])
>>> len(result)
1
>>> result[0]
array([0, 1, 2, 3, 4, 5, 6])
```
This is the simplest case. We have passed `[7]` as the `dimensions` argument, which represents a 1D array with 7 elements.\
For each dimension (so in this case just for one) the `result` gets a `[7]`-shaped subarray which is made of indices (0, 1, ...) counted only in the corresponding dimension.\
In the example there is only one dimension available, so the result only consists of one subarray with indices going along this dimension.

The next example is the most important one:
```python
>>> result = np.indices([2, 3])
>>> len(result)
2
>>> result[0]
array([[0, 0, 0],
       [1, 1, 1]])
>>> result[1]
array([[0, 1, 2],
       [0, 1, 2]])
```
Now, we have passed `[2, 3]` as the `dimensions` argument, which represents a 2D array with 2 rows and 3 columns.\
For each dimension (we have two of them here) the `result` gets a `[2, 3]`-shaped subarray made of indices counted only in the corresponding dimension. So the first subarray has indices counted along the rows, while the second one counts along the column axis.

That's it! We could go with higher dimensions, but for basic usage that's unnecessary.

### Use cases
`numpy.indices` is very useful when we want to define a matrix by its indices, which is common in maths/physics. For example: "A 3x5 matrix `M` where `M_ij = 2*i + j`" defines a matrix where each element is equal to its row index times two plus its column index.
```python
>>> import numpy as np
>>> i, j = np.indices([3, 5])
>>> M = 2 * i + j
>>> M
array([[0, 1, 2, 3, 4],
       [2, 3, 4, 5, 6],
       [4, 5, 6, 7, 8]])
```
Another interesting thing is that those indices can be treated as coordinates in space.\
That makes `numpy.indices` a simple choice for defining fields of values that depend on x, y (possibly z and more).
In maths/physics we call those fields scalar.
Unfortunately, the obvious limitation is that we can only deal with discrete coordinates.