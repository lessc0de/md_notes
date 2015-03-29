# Theano - Advanced Usage
## Manipulating Symbolic Expressions
### The "grad" method
```python
>>> x = T.scalar('x')
>>> y = 2. * x
>>> g = T.grad(y, x)
>>> from theano.printing import min_informative_str
>>> print min_informative_str(g)
A. Elemwise{mul}
 B. Elemwise{second,no_inplace}
  C. Elemwise{mul,no_inplace}
   D. TensorConstant{2.0}
   E. x
  F. TensorConstant{1.0}
 <D>
```

### Holding variables constant
Directly in the call to `grad`:

* `consider_constant` allows you to specify a list of variables to hold constant
  * flexible but easy to forget
* `block_gradient`
  * not in Theano, but in `pylearn2.utils`
  * makes a constant out of any expression
  * less flexible but no risk of forgetting

### Combing gradients from other sources
Suppose `z(x) = f(g(x))`. We want to split back-prop into 2 steps:

1. back-prop through `f`
2. back-prop through `g`

We do this for many reasons:

* Maybe `f` is not a Theano function
* We want to save memory

### `known_grads`
This is another parameter of the grad method. You specify a dictionary mapping variables to gradients that are already analytically known. You don't even need to specify a cost function if a separating set is already known.

## Theano Variables
A _variable_ is a theano expression. It can come from `T.scalar`, `T.matrix`, etc. It can also come from operations on other variables. Every variable has a "type" field that identifies its type (e.g. `TensorType((True, False), 'float32')`). Variables can be thought of as _nodes_ in a _graph_.

### Ops
An _op_ is any class that describes a mathematical function of some variables. You can call the op on some variables to get a new variable or variables. An op class can supply other forms of information about the function, such as its derivatives.

### Apply nodes
The _apply_ class is a specific instance of an application of an op. Notable fields include:

* `op`: the op to be applied
* `inputs`: the variables to be used as input
* `outputs`: the variables produced

`Variable.owner` identifies the Apply that created the variable. Variable and Apply instances are _nodes_ and owner/inputs/outputs identify _edges_ in a Theano graph.

