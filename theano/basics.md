# Theano Basics
## Configuring Theano
[Theano Configuration Options](http://deeplearning.net/software/theano/library/config.html#config-attributes)

* `~/.theanorc`: settings you always want
* `THEANO_FLAGS`: settings for one job
* `theano.config`: settings for right now

`.theanorc`:
```
[global]
device  = cpu
floatX  = float32

[warn]
argmax_pushdown_bug     = False
sum_div_dimshuffle_bug  = False
subtensor_merge_bug     = False
```

`THEANO_FLAGS`:
```
THEANO_FLAGS="floatX=float64" python my_amazing_theano_script.py
```

`theano.config`:
```python
>>> import theano
>>> theano.config.floatX = 'float32'
>>> x = theano.tensor.scalar()
>>> x.dtype
'float32'
>>> theano.config.floatX = 'float64'
>>> x = theano.tensor.scalar()
>>> x.dtype
`float64`
```

## Building Expressions
### Scalar
```python
from theano import tensor as T
x = T.scalar()
y = T.scalar()
z = x + y
w = z * x
a = T.sqrt(w)
b = T.exp(a)
c = a ** b
d = T.log(c)
```

### Vector
```python
from theano import tensor as T
x = T.vector()
y = T.vector()
# element-wise multiplication
a = x * y
# dot product
b = T.dot(x, y)
# broadcasting
c = a + b
```

### Matrix
```python
from theano import tensor as T
x = T.matrix()
y = T.matrix()
# matrix-matrix product
b = T.dot(x, y)
# matrix-vector product
a = T.vector()
c = T.dot(x, a)
```

### Tensor
Dimensionality is defined by the length of `Tensor`'s `broadcastable` argument. You can perform the add operation (or other element-wise operations) on two tensors with the same dimensionality.
```python
from theano import tensor as T
tensor3 = T.TensorType(broadcastable=(False, False, False), dtype='float32')
x = tensor3()
```

#### Reductions
```python
from theano import tensor as T
tensor3 = T.TensorType(broadcastable=(False, False, False), dtype='float32')
x = tensor3()

total = x.sum()
marginals = x.sum(axis=(0,2))
mx = x.max(axis=1)
```

#### Dimshuffle
```python
from theano import tensor as T
tensor3 = T.TensorType(broadcastable=(False, False, False), dtype='float32')
x = tensor3()

y = x.dimshuffle((2, 1, 0))
a = T.matrix()
b = a.T
# Same as b
c = a.dimshuffle((0, 1))
# Adding to a larger tensor
d = a.dimshuffle((0, 1, 'x'))  # add another dimension of size 1
e = a + d
```

#### `zeros_like` and `ones_like`

* `zeros_like(x)` returns a symbolic tensor with the same `shape` and `dtype` as `x`, but with every element equal to 0
* `ones_like(x)` is the same thing, but with 1s.


## Expression Compilation & Execution
### `theano.function`
```python
>>> from theano import tensor as T
>>> from theano import function
>>> x = T.scalar()
>>> y = T.scalar()
>>> # 1st arg is list of SYMBOLIC inputs
>>> # 2nd arg is SYMBOLIC output
>>> f = function([x, y], x + y)
>>> # Call w/ NUMERIC values
>>> # Get a NUMERIC output
>>> f(1., 2.)  # --> array(3.0)
array(3.0)
```

### Shared variables
It is hard to do much with just functional programming. Shared variables add a bit of imperative programming. A shared variable is a buffer that stores a numerical value for a Theano variable. You can write to a shared variable at the end of a function. Read/Write to a shared variable outside functional scope with `get_value` and `set_value`.

```python
>>> from theano import shared
>>> x = shared(0.)
>>> from theano.compat.python2x import OrderedDict
>>> updates = OrderedDict()
>>> updates[x] = x + 1
>>> f = function([], updates=updates)
>>> f()
[]
>>> x.get_value()
1.0
>>> x.set_value(100.)
f()
>>> x.get_value()
101.0
```

#### `theano.compat.python2x.OrderedDict`?
This dictionary is used as a mechanism for mutation. It is not

* `collections.OrderedDict`: this isn't available in older versions of Python
* `{}` or `dict`: the iteration order of this built-in class is not deterministic, so if Theano accepted this, the same script could compile different C programs each time you run it

### Compilation modes
You can compile in different modes to get different kinds of programs. These modes can be specified through arguments given to `theano.function` or through flags set in Theano's configuration.

* `mode=FAST_RUN`: [default] Spends a lot of time on compilation to get an executable that runs fast
* `mode=FAST_COMPILE`: Doesn't spend much time compiling. Executable uses Python instead of compiled C code
* `mode=DEBUG_MODE`: Adds a lot of checks. Raises error messages in situations other modes regard as passable/fine.

#### Compilation for GPU
Theano supports GPU compilation but only for 32-bit types. CUDA can support 64-bit, but it is slow. Set the device flag to "gpu" (or a specific gpu, like "gpu0").

* `T.fscalar`, `T.fvector`, `T.fmatrix` are all 32-bit
* `T.scalar`, `T.vector`, `T.matrix` resolve to 32-bit or 64-bit depending on Theano's "floatX" flag (64-bit by default)

### Optimizations
Theano changes the symbolic expressions you write before converting them to C code. These changes make them faster and more stable. For example:

* `(x + y) + (x + y)` --> `2 (x + y)`
* `exp(a)/exp(a).sum(axis = 1)` --> `softmax(a)`

Sometimes, optimizations discard error checking and produce incorrect output rather than throwing an exception.
```python
>>> x = T.scalar()
>>> f = function([x], x/x)
>>> f(0.)
array(1.0)
```

