<!-- This file is machine generated: DO NOT EDIT! -->

# Building Graphs
[TOC]

Classes and functions for building TensorFlow graphs.

## Core graph data structures

- - -

### `class tf.Graph` {#Graph}

A TensorFlow computation, represented as a dataflow graph.

A `Graph` contains a set of
[`Operation`](../../api_docs/python/framework.md#Operation) objects,
which represent units of computation; and
[`Tensor`](../../api_docs/python/framework.md#Tensor) objects, which represent
the units of data that flow between operations.

A default `Graph` is always registered, and accessible by calling
[`tf.get_default_graph()`](../../api_docs/python/framework.md#get_default_graph).
To add an operation to the default graph, simply call one of the functions
that defines a new `Operation`:

```
c = tf.constant(4.0)
assert c.graph is tf.get_default_graph()
```

Another typical usage involves the
[`Graph.as_default()`](../../api_docs/python/framework.md#Graph.as_default)
context manager, which overrides the current default graph for the
lifetime of the context:

```python
g = tf.Graph()
with g.as_default():
  # Define operations and tensors in `g`.
  c = tf.constant(30.0)
  assert c.graph is g
```

Important note: This class *is not* thread-safe for graph construction. All
operations should be created from a single thread, or external
synchronization must be provided. Unless otherwise specified, all methods
are not thread-safe.

- - -

#### `tf.Graph.__init__()` {#Graph.__init__}

Creates a new, empty Graph.


- - -

#### `tf.Graph.as_default()` {#Graph.as_default}

Returns a context manager that makes this `Graph` the default graph.

This method should be used if you want to create multiple graphs
in the same process. For convenience, a global default graph is
provided, and all ops will be added to this graph if you do not
create a new graph explicitly. Use this method the `with` keyword
to specify that ops created within the scope of a block should be
added to this graph.

The default graph is a property of the current thread. If you
create a new thread, and wish to use the default graph in that
thread, you must explicitly add a `with g.as_default():` in that
thread's function.

The following code examples are equivalent:

```python
# 1. Using Graph.as_default():
g = tf.Graph()
with g.as_default():
  c = tf.constant(5.0)
  assert c.graph is g

# 2. Constructing and making default:
with tf.Graph().as_default() as g:
  c = tf.constant(5.0)
  assert c.graph is g
```

##### Returns:

  A context manager for using this graph as the default graph.


- - -

#### `tf.Graph.as_graph_def(from_version=None)` {#Graph.as_graph_def}

Returns a serialized `GraphDef` representation of this graph.

The serialized `GraphDef` can be imported into another `Graph`
(using [`import_graph_def()`](#import_graph_def)) or used with the
[C++ Session API](../../api_docs/cc/index.md).

This method is thread-safe.

##### Args:


*  <b>`from_version`</b>: Optional.  If this is set, returns a `GraphDef`
    containing only the nodes that were added to this graph since
    its `version` property had the given value.

##### Returns:

  A [`GraphDef`](https://tensorflow.googlesource.com/tensorflow/+/master/tensorflow/core/framework/graph.proto)
  protocol buffer.

##### Raises:


*  <b>`ValueError`</b>: If the graph_def would be too large.


- - -

#### `tf.Graph.finalize()` {#Graph.finalize}

Finalizes this graph, making it read-only.

After calling `g.finalize()`, no new operations can be added to
`g`.  This method is used to ensure that no operations are added
to a graph when it is shared between multiple threads, for example
when using a [`QueueRunner`](../../api_docs/python/train.md#QueueRunner).


- - -

#### `tf.Graph.finalized` {#Graph.finalized}

True if this graph has been finalized.


- - -

#### `tf.Graph.control_dependencies(control_inputs)` {#Graph.control_dependencies}

Returns a context manager that specifies control dependencies.

Use with the `with` keyword to specify that all operations constructed
within the context should have control dependencies on
`control_inputs`. For example:

```python
with g.control_dependencies([a, b, c]):
  # `d` and `e` will only run after `a`, `b`, and `c` have executed.
  d = ...
  e = ...
```

Multiple calls to `control_dependencies()` can be nested, and in
that case a new `Operation` will have control dependencies on the union
of `control_inputs` from all active contexts.

```python
with g.control_dependencies([a, b]):
  # Ops declared here run after `a` and `b`.
  with g.control_dependencies([c, d]):
    # Ops declared here run after `a`, `b`, `c`, and `d`.
```

*N.B.* The control dependencies context applies *only* to ops that
are constructed within the context. Merely using an op or tensor
in the context does not add a control dependency. The following
example illustrates this point:

```python
# WRONG
def my_func(pred, tensor):
  t = tf.matmul(tensor, tensor)
  with tf.control_dependencies([pred]):
    # The matmul op is created outside the context, so no control
    # dependency will be added.
    return t

# RIGHT
def my_func(pred, tensor):
  with tf.control_dependencies([pred]):
    # The matmul op is created in the context, so a control dependency
    # will be added.
    return tf.matmul(tensor, tensor)
```

##### Args:


*  <b>`control_inputs`</b>: A list of `Operation` or `Tensor` objects, which
    must be executed or computed before running the operations
    defined in the context.

##### Returns:

 A context manager that specifies control dependencies for all
 operations constructed within the context.

##### Raises:


*  <b>`TypeError`</b>: If `control_inputs` is not a list of `Operation` or
    `Tensor` objects.


- - -

#### `tf.Graph.device(device_name_or_function)` {#Graph.device}

Returns a context manager that specifies the default device to use.

The `device_name_or_function` argument may either be a device name
string, a device function, or None:

* If it is a device name string, all operations constructed in
  this context will be assigned to the device with that name.
* If it is a function, it will be treated as function from
  Operation objects to device name strings, and invoked each time
  a new Operation is created. The Operation will be assigned to
  the device with the returned name.
* If it is None, the default device will be cleared.

For example:

```python
with g.device('/gpu:0'):
  # All operations constructed in this context will be placed
  # on GPU 0.
  with g.device(None):
    # All operations constructed in this context will have no
    # assigned device.

# Defines a function from `Operation` to device string.
def matmul_on_gpu(n):
  if n.type == "MatMul":
    return "/gpu:0"
  else:
    return "/cpu:0"

with g.device(matmul_on_gpu):
  # All operations of type "MatMul" constructed in this context
  # will be placed on GPU 0; all other operations will be placed
  # on CPU 0.
```

##### Args:


*  <b>`device_name_or_function`</b>: The device name or function to use in
    the context.

##### Returns:

  A context manager that specifies the default device to use for newly
  created ops.


- - -

#### `tf.Graph.name_scope(name)` {#Graph.name_scope}

Returns a context manager that creates hierarchical names for operations.

A graph maintains a stack of name scopes. A `with name_scope(...):`
statement pushes a new name onto the stack for the lifetime of the context.

The `name` argument will be interpreted as follows:

* A string (not ending with '/') will create a new name scope, in which
  `name` is appended to the prefix of all operations created in the
  context. If `name` has been used before, it will be made unique by
  calling `self.unique_name(name)`.
* A scope previously captured from a `with g.name_scope(...) as
  scope:` statement will be treated as an "absolute" name scope, which
  makes it possible to re-enter existing scopes.
* A value of `None` or the empty string will reset the current name scope
  to the top-level (empty) name scope.

For example:

```python
with tf.Graph().as_default() as g:
  c = tf.constant(5.0, name="c")
  assert c_1.name == "c"
  c_1 = tf.constant(6.0, name="c")
  assert c_1.name == "c_1"

  # Creates a scope called "nested"
  with g.name_scope("nested") as scope:
    nested_c = tf.constant(10.0, name="c")
    assert nested_c.name == "nested/c"

    # Creates a nested scope called "inner".
    with g.name_scope("inner"):
      nested_inner_c = tf.constant(20.0, name="c")
      assert nested_inner_c.name == "nested/inner/c"

    # Create a nested scope called "inner_1".
    with g.name_scope("inner"):
      nested_inner_1_c = tf.constant(30.0, name="c")
      assert nested_inner_1_c.name == "nested/inner_1/c"

      # Treats `scope` as an absolute name scope, and
      # switches to the "nested/" scope.
      with g.name_scope(scope):
        nested_d = tf.constant(40.0, name="d")
        assert nested_d.name == "nested/d"

        with g.name_scope(""):
          e = tf.constant(50.0, name="e")
          assert e.name == "e"
```

The name of the scope itself can be captured by `with
g.name_scope(...) as scope:`, which stores the name of the scope
in the variable `scope`. This value can be used to name an
operation that represents the overall result of executing the ops
in a scope. For example:

```python
inputs = tf.constant(...)
with g.name_scope('my_layer') as scope:
  weights = tf.Variable(..., name="weights")
  biases = tf.Variable(..., name="biases")
  affine = tf.matmul(inputs, weights) + biases
  output = tf.nn.relu(affine, name=scope)
```


##### Args:


*  <b>`name`</b>: A name for the scope.

##### Returns:

  A context manager that installs `name` as a new name scope.



A `Graph` instance supports an arbitrary number of "collections"
that are identified by name. For convenience when building a large
graph, collections can store groups of related objects: for
example, the `tf.Variable` uses a collection (named
[`tf.GraphKeys.VARIABLES`](../../api_docs/python/framework.md#GraphKeys)) for
all variables that are created during the construction of a graph. The caller
may define additional collections by specifying a new name.

- - -

#### `tf.Graph.add_to_collection(name, value)` {#Graph.add_to_collection}

Stores `value` in the collection with the given `name`.

##### Args:


*  <b>`name`</b>: The key for the collection. For example, the `GraphKeys` class
    contains many standard names for collections.
*  <b>`value`</b>: The value to add to the collection.


- - -

#### `tf.Graph.get_collection(name, scope=None)` {#Graph.get_collection}

Returns a list of values in the collection with the given `name`.

##### Args:


*  <b>`key`</b>: The key for the collection. For example, the `GraphKeys` class
    contains many standard names for collections.
*  <b>`scope`</b>: (Optional.) If supplied, the resulting list is filtered to include
    only items whose name begins with this string.

##### Returns:

  The list of values in the collection with the given `name`, or
  an empty list if no value has been added to that collection. The
  list contains the values in the order under which they were
  collected.



- - -

#### `tf.Graph.as_graph_element(obj, allow_tensor=True, allow_operation=True)` {#Graph.as_graph_element}

Returns the object referred to by `obj`, as an `Operation` or `Tensor`.

This function validates that `obj` represents an element of this
graph, and gives an informative error message if it is not.

This function is the canonical way to get/validate an object of
one of the allowed types from an external argument reference in the
Session API.

This method may be called concurrently from multiple threads.

##### Args:


*  <b>`obj`</b>: A `Tensor`, an `Operation`, or the name of a tensor or operation.
    Can also be any object with an `_as_graph_element()` method that returns
    a value of one of these types.
*  <b>`allow_tensor`</b>: If true, `obj` may refer to a `Tensor`.
*  <b>`allow_operation`</b>: If true, `obj` may refer to an `Operation`.

##### Returns:

  The `Tensor` or `Operation` in the Graph corresponding to `obj`.

##### Raises:


*  <b>`TypeError`</b>: If `obj` is not a type we support attempting to convert
    to types.
*  <b>`ValueError`</b>: If `obj` is of an appropriate type but invalid. For
    example, an invalid string.
*  <b>`KeyError`</b>: If `obj` is not an object in the graph.


- - -

#### `tf.Graph.get_operation_by_name(name)` {#Graph.get_operation_by_name}

Returns the `Operation` with the given `name`.

This method may be called concurrently from multiple threads.

##### Args:


*  <b>`name`</b>: The name of the `Operation` to return.

##### Returns:

  The `Operation` with the given `name`.

##### Raises:


*  <b>`TypeError`</b>: If `name` is not a string.
*  <b>`KeyError`</b>: If `name` does not correspond to an operation in this graph.


- - -

#### `tf.Graph.get_tensor_by_name(name)` {#Graph.get_tensor_by_name}

Returns the `Tensor` with the given `name`.

This method may be called concurrently from multiple threads.

##### Args:


*  <b>`name`</b>: The name of the `Tensor` to return.

##### Returns:

  The `Tensor` with the given `name`.

##### Raises:


*  <b>`TypeError`</b>: If `name` is not a string.
*  <b>`KeyError`</b>: If `name` does not correspond to a tensor in this graph.


- - -

#### `tf.Graph.get_operations()` {#Graph.get_operations}

Return the list of operations in the graph.

You can modify the operations in place, but modifications
to the list such as inserts/delete have no effect on the
list of operations known to the graph.

This method may be called concurrently from multiple threads.

##### Returns:

  A list of Operations.



- - -

#### `tf.Graph.get_default_device()` {#Graph.get_default_device}

Returns the default device.

##### Returns:

  A string.


- - -

#### `tf.Graph.seed` {#Graph.seed}



- - -

#### `tf.Graph.unique_name(name)` {#Graph.unique_name}

Return a unique operation name for `name`.

Note: You rarely need to call `unique_name()` directly.  Most of
the time you just need to create `with g.name_scope()` blocks to
generate structured names.

`unique_name` is used to generate structured names, separated by
`"/"`, to help identify operations when debugging a graph.
Operation names are displayed in error messages reported by the
TensorFlow runtime, and in various visualization tools such as
TensorBoard.

##### Args:


*  <b>`name`</b>: The name for an operation.

##### Returns:

  A string to be passed to `create_op()` that will be used
  to name the operation being created.


- - -

#### `tf.Graph.version` {#Graph.version}

Returns a version number that increases as ops are added to the graph.


- - -

#### `tf.Graph.create_op(op_type, inputs, dtypes, input_types=None, name=None, attrs=None, op_def=None, compute_shapes=True)` {#Graph.create_op}

Creates an `Operation` in this graph.

This is a low-level interface for creating an `Operation`. Most
programs will not call this method directly, and instead use the
Python op constructors, such as `tf.constant()`, which add ops to
the default graph.

##### Args:


*  <b>`op_type`</b>: The `Operation` type to create. This corresponds to the
    `OpDef.name` field for the proto that defines the operation.
*  <b>`inputs`</b>: A list of `Tensor` objects that will be inputs to the `Operation`.
*  <b>`dtypes`</b>: A list of `DType` objects that will be the types of the tensors
    that the operation produces.
*  <b>`input_types`</b>: (Optional.) A list of `DType`s that will be the types of
    the tensors that the operation consumes. By default, uses the base
    `DType` of each input in `inputs`. Operations that expect
    reference-typed inputs must specify `input_types` explicitly.
*  <b>`name`</b>: (Optional.) A string name for the operation. If not specified, a
    name is generated based on `op_type`.
*  <b>`attrs`</b>: (Optional.) A list of `AttrValue` protos for the `attr` field of
    the `NodeDef` proto that will represent the operation.
*  <b>`op_def`</b>: (Optional.) The `OpDef` proto that describes the `op_type` that
    the operation will have.
*  <b>`compute_shapes`</b>: (Optional.) If True, shape inference will be performed
    to compute the shapes of the outputs.

##### Raises:


*  <b>`TypeError`</b>: if any of the inputs is not a `Tensor`.

##### Returns:

  An `Operation` object.


- - -

#### `tf.Graph.gradient_override_map(op_type_map)` {#Graph.gradient_override_map}

EXPERIMENTAL: A context manager for overriding gradient functions.

This context manager can be used to override the gradient function
that will be used for ops within the scope of the context.

For example:

```python
@tf.RegisterGradient("CustomSquare")
def _custom_square_grad(op, inputs):
  # ...

with tf.Graph().as_default() as g:
  c = tf.constant(5.0)
  s_1 = tf.square(c)  # Uses the default gradient for tf.square.
  with g.gradient_override_map({"Square": "CustomSquare"}):
    s_2 = tf.square(s_2)  # Uses _custom_square_grad to compute the
                          # gradient of s_2.
```

##### Args:


*  <b>`op_type_map`</b>: A dictionary mapping op type strings to alternative op
    type strings.

##### Returns:

  A context manager that sets the alternative op type to be used for one
  or more ops created in that context.

##### Raises:


*  <b>`TypeError`</b>: If `op_type_map` is not a dictionary mapping strings to
    strings.



- - -

### `class tf.Operation` {#Operation}

Represents a graph node that performs computation on tensors.

An `Operation` is a node in a TensorFlow `Graph` that takes zero or
more `Tensor` objects as input, and produces zero or more `Tensor`
objects as output. Objects of type `Operation` are created by
calling a Python op constructor (such as
[`tf.matmul()`](../../api_docs/python/math_ops.md#matmul))
or [`Graph.create_op()`](../../api_docs/python/framework.md#Graph.create_op).

For example `c = tf.matmul(a, b)` creates an `Operation` of type
"MatMul" that takes tensors `a` and `b` as input, and produces `c`
as output.

After the graph has been launched in a session, an `Operation` can
be executed by passing it to
[`Session.run()`](../../api_docs/python/client.md#Session.run).
`op.run()` is a shortcut for calling `tf.get_default_session().run(op)`.

- - -

#### `tf.Operation.name` {#Operation.name}

The full name of this operation.

- - -

#### `tf.Operation.type` {#Operation.type}

The type of the op (e.g. `"MatMul"`).

- - -

#### `tf.Operation.inputs` {#Operation.inputs}

The list of `Tensor` objects representing the data inputs of this op.

- - -

#### `tf.Operation.control_inputs` {#Operation.control_inputs}

The `Operation` objects on which this op has a control dependency.

Before this op is executed, TensorFlow will ensure that the
operations in `self.control_inputs` have finished executing. This
mechanism can be used to run ops sequentially for performance
reasons, or to ensure that the side effects of an op are observed
in the correct order.

##### Returns:

  A list of `Operation` objects.

- - -

#### `tf.Operation.outputs` {#Operation.outputs}

The list of `Tensor` objects representing the outputs of this op.

- - -

#### `tf.Operation.device` {#Operation.device}

The name of the device to which this op has been assigned, if any.

##### Returns:

  The string name of the device to which this op has been
  assigned, or None if it has not been assigned to a device.

- - -

#### `tf.Operation.graph` {#Operation.graph}

The `Graph` that contains this operation.


- - -

#### `tf.Operation.run(feed_dict=None, session=None)` {#Operation.run}

Runs this operation in a `Session`.

Calling this method will execute all preceding operations that
produce the inputs needed for this operation.

*N.B.* Before invoking `Operation.run()`, its graph must have been
launched in a session, and either a default session must be
available, or `session` must be specified explicitly.

##### Args:


*  <b>`feed_dict`</b>: A dictionary that maps `Tensor` objects to feed values.
    See [`Session.run()`](../../api_docs/python/client.md#Session.run)
    for a description of the valid feed values.
*  <b>`session`</b>: (Optional.) The `Session` to be used to run to this operation. If
    none, the default session will be used.



- - -

#### `tf.Operation.get_attr(name)` {#Operation.get_attr}

Returns the value of the attr of this op with the given `name`.

##### Args:


*  <b>`name`</b>: The name of the attr to fetch.

##### Returns:

  The value of the attr, as a Python object.

##### Raises:


*  <b>`ValueError`</b>: If this op does not have an attr with the given `name`.


- - -

#### `tf.Operation.traceback` {#Operation.traceback}

Returns the call stack from when this operation was constructed.


#### Other Methods
- - -

#### `tf.Operation.__init__(node_def, g, inputs=None, output_types=None, control_inputs=None, input_types=None, original_op=None, op_def=None)` {#Operation.__init__}

Creates an `Operation`.

NOTE: This constructor validates the name of the `Operation` (passed
as `node_def.name`). Valid `Operation` names match the following
regular expression:

    [A-Za-z0-9.][A-Za-z0-9_.\-/]*

##### Args:


*  <b>`node_def`</b>: `graph_pb2.NodeDef`.  `NodeDef` for the `Operation`.
    Used for attributes of `graph_pb2.NodeDef`, typically `name`,
    `op`, and `device`.  The `input` attribute is irrelevant here
    as it will be computed when generating the model.
*  <b>`g`</b>: `Graph`. The parent graph.
*  <b>`inputs`</b>: list of `Tensor` objects. The inputs to this `Operation`.
*  <b>`output_types`</b>: list of `DType` objects.  List of the types of the
    `Tensors` computed by this operation.  The length of this list indicates
    the number of output endpoints of the `Operation`.
*  <b>`control_inputs`</b>: list of operations or tensors from which to have a
    control dependency.
*  <b>`input_types`</b>: List of `DType` objects representing the
    types of the tensors accepted by the `Operation`.  By default
    uses `[x.dtype.base_dtype for x in inputs]`.  Operations that expect
    reference-typed inputs must specify these explicitly.
*  <b>`original_op`</b>: Optional. Used to associate the new `Operation` with an
    existing `Operation` (for example, a replica with the op that was
    replicated).
*  <b>`op_def`</b>: Optional. The `op_def_pb2.OpDef` proto that describes the
    op type that this `Operation` represents.

##### Raises:


*  <b>`TypeError`</b>: if control inputs are not Operations or Tensors,
    or if node_def is not a `NodeDef`,
    or if g is not a `Graph`,
    or if inputs are not tensors,
    or if inputs and input_types are incompatible.
*  <b>`ValueError`</b>: if the node_def name is not valid.


- - -

#### `tf.Operation.node_def` {#Operation.node_def}

Returns a serialized `NodeDef` representation of this operation.

##### Returns:

  A
  [`NodeDef`](https://tensorflow.googlesource.com/tensorflow/+/master/tensorflow/core/framework/graph.proto)
  protocol buffer.

- - -

#### `tf.Operation.op_def` {#Operation.op_def}

Returns the `OpDef` proto that represents the type of this op.

##### Returns:

  An
  [`OpDef`](https://tensorflow.googlesource.com/tensorflow/+/master/tensorflow/core/framework/op_def.proto)
  protocol buffer.

- - -

#### `tf.Operation.values()` {#Operation.values}

DEPRECATED: Use outputs.



- - -

### `class tf.Tensor` {#Tensor}

Represents a value produced by an `Operation`.

A `Tensor` is a symbolic handle to one of the outputs of an
`Operation`. It does not hold the values of that operation's output,
but instead provides a means of computing those values in a
TensorFlow [`Session`](../../api_docs/python/client.md#Session).

This class has two primary purposes:

1. A `Tensor` can be passed as an input to another `Operation`.
   This builds a dataflow connection between operations, which
   enables TensorFlow to execute an entire `Graph` that represents a
   large, multi-step computation.

2. After the graph has been launched in a session, the value of the
   `Tensor` can be computed by passing it to
   [`Session.run()`](../../api_docs/python/client.md#Session.run).
   `t.eval()` is a shortcut for calling
   `tf.get_default_session().run(t)`.

In the following example, `c`, `d`, and `e` are symbolic `Tensor`
objects, whereas `result` is a numpy array that stores a concrete
value:

```python
# Build a dataflow graph.
c = tf.constant([[1.0, 2.0], [3.0, 4.0]])
d = tf.constant([[1.0, 1.0], [0.0, 1.0]])
e = tf.matmul(c, d)

# Construct a `Session` to execut the graph.
sess = tf.Session()

# Execute the graph and store the value that `e` represents in `result`.
result = sess.run(e)
```

- - -

#### `tf.Tensor.dtype` {#Tensor.dtype}

The `DType` of elements in this tensor.

- - -

#### `tf.Tensor.name` {#Tensor.name}

The string name of this tensor.

- - -

#### `tf.Tensor.value_index` {#Tensor.value_index}

The index of this tensor in the outputs of its `Operation`.

- - -

#### `tf.Tensor.graph` {#Tensor.graph}

The `Graph` that contains this tensor.

- - -

#### `tf.Tensor.op` {#Tensor.op}

The `Operation` that produces this tensor as an output.

- - -

#### `tf.Tensor.consumers()` {#Tensor.consumers}

Returns a list of `Operation`s that consume this tensor.

##### Returns:

  A list of `Operation`s.



- - -

#### `tf.Tensor.eval(feed_dict=None, session=None)` {#Tensor.eval}

Evaluates this tensor in a `Session`.

Calling this method will execute all preceding operations that
produce the inputs needed for the operation that produces this
tensor.

*N.B.* Before invoking `Tensor.eval()`, its graph must have been
launched in a session, and either a default session must be
available, or `session` must be specified explicitly.

##### Args:


*  <b>`feed_dict`</b>: A dictionary that maps `Tensor` objects to feed values.
    See [`Session.run()`](../../api_docs/python/client.md#Session.run) for a
    description of the valid feed values.
*  <b>`session`</b>: (Optional.) The `Session` to be used to evaluate this tensor. If
    none, the default session will be used.

##### Returns:

  A numpy array corresponding to the value of this tensor.



- - -

#### `tf.Tensor.get_shape()` {#Tensor.get_shape}

Returns the `TensorShape` that represents the shape of this tensor.

The shape is computed using shape inference functions that are
registered for each `Operation` type using `tf.RegisterShape`.
See [`TensorShape`](../../api_docs/python/framework.md#TensorShape) for more
details of what a shape represents.

The inferred shape of a tensor is used to provide shape
information without having to launch the graph in a session. This
can be used for debugging, and providing early error messages. For
example:

```python
c = tf.constant([[1.0, 2.0, 3.0], [4.0, 5.0, 6.0]])

print c.get_shape()
==> TensorShape([Dimension(2), Dimension(3)])

d = tf.constant([[1.0, 0.0], [0.0, 1.0], [1.0, 0.0], [0.0, 1.0]])

print d.get_shape()
==> TensorShape([Dimension(4), Dimension(2)])

# Raises a ValueError, because `c` and `d` do not have compatible
# inner dimensions.
e = tf.matmul(c, d)

f = tf.matmul(c, d, transpose_a=True, transpose_b=True)

print f.get_shape()
==> TensorShape([Dimension(3), Dimension(4)])
```

In some cases, the inferred shape may have unknown dimensions. If
the caller has additional information about the values of these
dimensions, `Tensor.set_shape()` can be used to augment the
inferred shape.

##### Returns:

  A `TensorShape` representing the shape of this tensor.


- - -

#### `tf.Tensor.set_shape(shape)` {#Tensor.set_shape}

Updates the shape of this tensor.

This method can be called multiple times, and will merge the given
`shape` with the current shape of this tensor. It can be used to
provide additional information about the shape of this tensor that
cannot be inferred from the graph alone. For example, this can be used
to provide additional information about the shapes of images:

```python
_, image_data = tf.TFRecordReader(...).read(...)
image = tf.image.decode_png(image_data, channels=3)

# The height and width dimensions of `image` are data dependent, and
# cannot be computed without executing the op.
print image.get_shape()
==> TensorShape([Dimension(None), Dimension(None), Dimension(3)])

# We know that each image in this dataset is 28 x 28 pixels.
image.set_shape([28, 28, 3])
print image.get_shape()
==> TensorShape([Dimension(28), Dimension(28), Dimension(3)])
```

##### Args:


*  <b>`shape`</b>: A `TensorShape` representing the shape of this tensor.

##### Raises:


*  <b>`ValueError`</b>: If `shape` is not compatible with the current shape of
    this tensor.



#### Other Methods
- - -

#### `tf.Tensor.__init__(op, value_index, dtype)` {#Tensor.__init__}

Creates a new `Tensor`.

##### Args:


*  <b>`op`</b>: An `Operation`. `Operation` that computes this tensor.
*  <b>`value_index`</b>: An `int`. Index of the operation's endpoint that produces
    this tensor.
*  <b>`dtype`</b>: A `DType`. Type of elements stored in this tensor.

##### Raises:


*  <b>`TypeError`</b>: If the op is not an `Operation`.


- - -

#### `tf.Tensor.device` {#Tensor.device}

The name of the device on which this tensor will be produced, or None.



## Tensor types

- - -

### `class tf.DType` {#DType}

Represents the type of the elements in a `Tensor`.

The following `DType` objects are defined:

* `tf.float32`: 32-bit single-precision floating-point.
* `tf.float64`: 64-bit double-precision floating-point.
* `tf.bfloat16`: 16-bit truncated floating-point.
* `tf.complex64`: 64-bit single-precision complex.

* `tf.int8`: 8-bit signed integer.
* `tf.uint8`: 8-bit unsigned integer.
* `tf.int32`: 32-bit signed integer.
* `tf.int64`: 64-bit signed integer.

* `tf.bool`: Boolean.

* `tf.string`: String.

* `tf.qint8`: Quantized 8-bit signed integer.
* `tf.quint8`: Quantized 8-bit unsigned integer.
* `tf.qint32`: Quantized 32-bit signed integer.

In addition, variants of these types with the `_ref` suffix are
defined for reference-typed tensors.

The `tf.as_dtype()` function converts numpy types and string type
names to a `DType` object.

- - -

#### `tf.DType.is_compatible_with(other)` {#DType.is_compatible_with}

Returns True if the `other` DType will be converted to this DType.

The conversion rules are as follows:

```
DType(T)       .is_compatible_with(DType(T))        == True
DType(T)       .is_compatible_with(DType(T).as_ref) == True
DType(T).as_ref.is_compatible_with(DType(T))        == False
DType(T).as_ref.is_compatible_with(DType(T).as_ref) == True
```

##### Args:


*  <b>`other`</b>: A `DType` (or object that may be converted to a `DType`).

##### Returns:

  True if a Tensor of the `other` `DType` will be implicitly converted to
  this `DType`.


- - -

#### `tf.DType.name` {#DType.name}

Returns the string name for this `DType`.

- - -

#### `tf.DType.base_dtype` {#DType.base_dtype}

Returns a non-reference `DType` based on this `DType`.

- - -

#### `tf.DType.is_ref_dtype` {#DType.is_ref_dtype}

Returns `True` if this `DType` represents a reference type.

- - -

#### `tf.DType.as_ref` {#DType.as_ref}

Returns a reference `DType` based on this `DType`.

- - -

#### `tf.DType.is_integer` {#DType.is_integer}

Returns whether this is a (non-quantized) integer type.

- - -

#### `tf.DType.is_quantized` {#DType.is_quantized}

Returns whether this is a quantized data type.


- - -

#### `tf.DType.as_numpy_dtype` {#DType.as_numpy_dtype}

Returns a `numpy.dtype` based on this `DType`.

- - -

#### `tf.DType.as_datatype_enum` {#DType.as_datatype_enum}

Returns a `types_pb2.DataType` enum value based on this `DType`.


#### Other Methods
- - -

#### `tf.DType.__init__(type_enum)` {#DType.__init__}

Creates a new `DataType`.

NOTE(mrry): In normal circumstances, you should not need to
construct a DataType object directly. Instead, use the
types.as_dtype() function.

##### Args:


*  <b>`type_enum`</b>: A `types_pb2.DataType` enum value.

##### Raises:


*  <b>`TypeError`</b>: If `type_enum` is not a value `types_pb2.DataType`.


- - -

#### `tf.DType.is_floating` {#DType.is_floating}

Returns whether this is a (real) floating point type.

- - -

#### `tf.DType.max` {#DType.max}

Returns the maximum representable value in this data type.

##### Raises:


*  <b>`TypeError`</b>: if this is a non-numeric, unordered, or quantized type.

- - -

#### `tf.DType.min` {#DType.min}

Returns the minimum representable value in this data type.

##### Raises:


*  <b>`TypeError`</b>: if this is a non-numeric, unordered, or quantized type.


- - -

### `tf.as_dtype(type_value)` {#as_dtype}

Converts the given `type_value` to a `DType`.

##### Args:


*  <b>`type_value`</b>: A value that can be converted to a `tf.DType`
    object. This may currently be a `tf.DType` object, a
    [`DataType` enum](https://tensorflow.googlesource.com/tensorflow/+/master/tensorflow/core/framework/types.proto),
    a string type name, or a `numpy.dtype`.

##### Returns:

  A `DType` corresponding to `type_value`.

##### Raises:


*  <b>`TypeError`</b>: If `type_value` cannot be converted to a `DType`.



## Utility functions

- - -

### `tf.device(dev)` {#device}

Wrapper for `Graph.device()` using the default graph.

See
[`Graph.name_scope()`](../../api_docs/python/framework.md#Graph.name_scope)
for more details.

##### Args:


*  <b>`device_name_or_function`</b>: The device name or function to use in
    the context.

##### Returns:

  A context manager that specifies the default device to use for newly
  created ops.


- - -

### `tf.name_scope(name)` {#name_scope}

Wrapper for `Graph.name_scope()` using the default graph.

See
[`Graph.name_scope()`](../../api_docs/python/framework.md#Graph.name_scope)
for more details.

##### Args:


*  <b>`name`</b>: A name for the scope.

##### Returns:

  A context manager that installs `name` as a new name scope in the
  default graph.


- - -

### `tf.control_dependencies(control_inputs)` {#control_dependencies}

Wrapper for `Graph.control_dependencies()` using the default graph.

See [`Graph.control_dependencies()`](../../api_docs/python/framework.md#Graph.control_dependencies)
for more details.

##### Args:


*  <b>`control_inputs`</b>: A list of `Operation` or `Tensor` objects, which
    must be executed or computed before running the operations
    defined in the context.

##### Returns:

 A context manager that specifies control dependencies for all
 operations constructed within the context.


- - -

### `tf.convert_to_tensor(value, dtype=None, name=None)` {#convert_to_tensor}

Converts the given `value` to a `Tensor`.

This function converts Python objects of various types to `Tensor`
objects. It accepts `Tensor` objects, numpy arrays, Python lists,
and Python scalars. For example:

```python
import numpy as np
array = np.random.rand((32, 100, 100))

def my_func(arg):
  arg = tf.convert_to_tensor(arg, dtype=tf.float32)
  return tf.matmul(arg, arg) + arg

# The following calls are equivalent.
value_1 = my_func(tf.constant([[1.0, 2.0], [3.0, 4.0]]))
value_2 = my_func([[1.0, 2.0], [3.0, 4.0]])
value_3 = my_func(np.array([[1.0, 2.0], [3.0, 4.0]], dtype=np.float32))
```

This function can be useful when composing a new operation in Python
(such as `my_func` in the example above). All standard Python op
constructors apply this function to each of their Tensor-valued
inputs, which allows those ops to accept numpy arrays, Python lists,
and scalars in addition to `Tensor` objects.

##### Args:


*  <b>`value`</b>: An object whose type has a registered `Tensor` conversion function.
*  <b>`dtype`</b>: Optional element type for the returned tensor. If missing, the
    type is inferred from the type of `value`.
*  <b>`name`</b>: Optional name to use if a new `Tensor` is created.

##### Returns:

  A `Tensor` based on `value`.

##### Raises:


*  <b>`TypeError`</b>: If no conversion function is registered for `value`.
*  <b>`RuntimeError`</b>: If a registered conversion function returns an invalid value.


- - -

### `tf.get_default_graph()` {#get_default_graph}

Returns the default graph for the current thread.

The returned graph will be the innermost graph on which a
`Graph.as_default()` context has been entered, or a global default
graph if none has been explicitly created.

*N.B.* The default graph is a property of the current thread. If you
create a new thread, and wish to use the default graph in that
thread, you must explicitly add a `with g.as_default():` in that
thread's function.

##### Returns:

  The default `Graph` being used in the current thread.


- - -

### `tf.import_graph_def(graph_def, input_map=None, return_elements=None, name=None, op_dict=None)` {#import_graph_def}

Imports the TensorFlow graph in `graph_def` into the Python `Graph`.

This function provides a way to import a serialized TensorFlow
[`GraphDef`](https://tensorflow.googlesource.com/tensorflow/+/master/tensorflow/core/framework/graph.proto)
protocol buffer, and extract individual objects in the `GraphDef` as
[`Tensor`](#Tensor) and [`Operation`](#Operation) objects. See
[`Graph.as_graph_def()`](#Graph.as_graph_def) for a way to create a
`GraphDef` proto.

##### Args:


*  <b>`graph_def`</b>: A `GraphDef` proto containing operations to be imported into
    the default graph.
*  <b>`input_map`</b>: A dictionary mapping input names (as strings) in `graph_def`
    to `Tensor` objects. The values of the named input tensors in the
    imported graph will be re-mapped to the respective `Tensor` values.
*  <b>`return_elements`</b>: A list of strings containing operation names in
    `graph_def` that will be returned as `Operation` objects; and/or
    tensor names in `graph_def` that will be returned as `Tensor` objects.
*  <b>`name`</b>: (Optional.) A prefix that will be prepended to the names in
    `graph_def`. Defaults to `"import"`.
*  <b>`op_dict`</b>: (Optional.) A dictionary mapping op type names to `OpDef` protos.
    Must contain an `OpDef` proto for each op type named in `graph_def`.
    If omitted, uses the `OpDef` protos registered in the global registry.

##### Returns:

  A list of `Operation` and/or `Tensor` objects from the imported graph,
  corresponding to the names in `return_elements'.

##### Raises:


*  <b>`TypeError`</b>: If `graph_def` is not a `GraphDef` proto,
    `input_map' is not a dictionary mapping strings to `Tensor` objects,
    or `return_elements` is not a list of strings.
*  <b>`ValueError`</b>: If `input_map`, or `return_elements` contains names that
    do not appear in `graph_def`, or `graph_def` is not well-formed (e.g.
    it refers to an unknown tensor).



## Graph collections

- - -

### `tf.add_to_collection(name, value)` {#add_to_collection}

Wrapper for `Graph.add_to_collection()` using the default graph.

See [`Graph.add_to_collection()`](../../api_docs/python/framework.md#Graph.add_to_collection)
for more details.

##### Args:


*  <b>`name`</b>: The key for the collection. For example, the `GraphKeys` class
    contains many standard names for collections.
*  <b>`value`</b>: The value to add to the collection.


- - -

### `tf.get_collection(key, scope=None)` {#get_collection}

Wrapper for `Graph.get_collection()` using the default graph.

See [`Graph.get_collection()`](../../api_docs/python/framework.md#Graph.get_collection)
for more details.

##### Args:


*  <b>`key`</b>: The key for the collection. For example, the `GraphKeys` class
    contains many standard names for collections.
*  <b>`scope`</b>: (Optional.) If supplied, the resulting list is filtered to include
    only items whose name begins with this string.

##### Returns:

  The list of values in the collection with the given `name`, or
  an empty list if no value has been added to that collection. The
  list contains the values in the order under which they were
  collected.


- - -

### `class tf.GraphKeys` {#GraphKeys}

Standard names to use for graph collections.

The standard library uses various well-known names to collect and
retrieve values associated with a graph. For example, the
`tf.Optimizer` subclasses default to optimizing the variables
collected under `tf.GraphKeys.TRAINABLE_VARIABLES` if none is
specified, but it is also possible to pass an explicit list of
variables.

The following standard keys are defined:

* `VARIABLES`: the `Variable` objects that comprise a model, and
  must be saved and restored together. See
  [`tf.all_variables()`](../../api_docs/python/state_ops.md#all_variables)
  for more details.
* `TRAINABLE_VARIABLES`: the subset of `Variable` objects that will
  be trained by an optimizer. See
  [`tf.trainable_variables()`](../../api_docs/python/state_ops.md#trainable_variables)
  for more details.
* `SUMMARIES`: the summary `Tensor` objects that have been created in the
  graph. See
  [`tf.merge_all_summaries()`](../../api_docs/python/train.md#merge_all_summaries)
  for more details.
* `QUEUE_RUNNERS`: the `QueueRunner` objects that are used to
  produce input for a computation. See
  [`tf.start_queue_runners()`](../../api_docs/python/train.md#start_queue_runners)
  for more details.


## Defining new operations

- - -

### `class tf.RegisterGradient` {#RegisterGradient}

A decorator for registering the gradient function for an op type.

This decorator is only used when defining a new op type. For an op
with `m` inputs and `n` inputs, the gradient function is a function
that takes the original `Operation` and `n` `Tensor` objects
(representing the gradients with respect to each output of the op),
and returns `m` `Tensor` objects (representing the partial gradients
with respect to each input of the op).

For example, assuming that operations of type `"Sub"` take two
inputs `x` and `y`, and return a single output `x - y`, the
following gradient function would be registered:

```python
@tf.RegisterGradient("Sub")
def _sub_grad(unused_op, grad):
  return grad, tf.Neg(grad)
```

The decorator argument `op_type` is the string type of an
operation. This corresponds to the `OpDef.name` field for the proto
that defines the operation.

- - -

#### `tf.RegisterGradient.__init__(op_type)` {#RegisterGradient.__init__}

Creates a new decorator with `op_type` as the Operation type.

##### Args:


*  <b>`op_type`</b>: The string type of an operation. This corresponds to the
    `OpDef.name` field for the proto that defines the operation.



- - -

### `tf.NoGradient(op_type)` {#NoGradient}

Specifies that ops of type `op_type` do not have a defined gradient.

This function is only used when defining a new op type. It may be
used for ops such as `tf.size()` that are not differentiable.  For
example:

```python
tf.NoGradient("Size")
```

##### Args:


*  <b>`op_type`</b>: The string type of an operation. This corresponds to the
    `OpDef.name` field for the proto that defines the operation.

##### Raises:


*  <b>`TypeError`</b>: If `op_type` is not a string.


- - -

### `class tf.RegisterShape` {#RegisterShape}

A decorator for registering the shape function for an op type.

This decorator is only used when defining a new op type. A shape
function is a function from an `Operation` object to a list of
`TensorShape` objects, with one `TensorShape` for each output of the
operation.

For example, assuming that operations of type `"Sub"` take two
inputs `x` and `y`, and return a single output `x - y`, all with the
same shape, the following shape function would be registered:

```python
@tf.RegisterShape("Sub")
def _sub_shape(op):
  return [op.inputs[0].get_shape().merge_with(op.inputs[1].get_shape())]
```

The decorator argument `op_type` is the string type of an
operation. This corresponds to the `OpDef.name` field for the proto
that defines the operation.
- - -

#### `tf.RegisterShape.__init__(op_type)` {#RegisterShape.__init__}

Saves the "op_type" as the Operation type.



- - -

### `class tf.TensorShape` {#TensorShape}

Represents the shape of a `Tensor`.

A `TensorShape` represents a possibly-partial shape specification for a
`Tensor`. It may be one of the following:

* *Fully-known shape:* has a known number of dimensions and a known size
  for each dimension.
* *Partially-known shape:* has a known number of dimensions, and an unknown
  size for one or more dimension.
* *Unknown shape:* has an unknown number of dimensions, and an unknown
  size in all dimensions.

If a tensor is produced by an operation of type `"Foo"`, its shape
may be inferred if there is a registered shape function for
`"Foo"`. See [`tf.RegisterShape()`](../../api_docs/python/framework.md#RegisterShape)
for details of shape
functions and how to register them. Alternatively, the shape may be set
explicitly using [`Tensor.set_shape()`](../../api_docs/python/framework.md#Tensor.set_shape).

- - -

#### `tf.TensorShape.merge_with(other)` {#TensorShape.merge_with}

Returns a `TensorShape` combining the information in `self` and `other`.

The dimensions in `self` and `other` are merged elementwise,
according to the rules defined for `Dimension.merge_with()`.

##### Args:


*  <b>`other`</b>: Another `TensorShape`.

##### Returns:

  A `TensorShape` containing the combined information of `self` and
  `other`.

##### Raises:


*  <b>`ValueError`</b>: If `self` and `other` are not compatible.


- - -

#### `tf.TensorShape.concatenate(other)` {#TensorShape.concatenate}

Returns the concatenation of the dimension in `self` and `other`.

*N.B.* If either `self` or `other` is completely unknown,
concatenation will discard information about the other shape. In
future, we might support concatenation that preserves this
information for use with slicing.

##### Args:


*  <b>`other`</b>: Another `TensorShape`.

##### Returns:

  A `TensorShape` whose dimensions are the concatenation of the
  dimensions in `self` and `other`.



- - -

#### `tf.TensorShape.ndims` {#TensorShape.ndims}

Returns the rank of this shape, or None if it is unspecified.

- - -

#### `tf.TensorShape.dims` {#TensorShape.dims}

Returns a list of Dimensions, or None if the shape is unspecified.

- - -

#### `tf.TensorShape.as_list()` {#TensorShape.as_list}

Returns a list of integers or None for each dimension.


- - -

#### `tf.TensorShape.is_compatible_with(other)` {#TensorShape.is_compatible_with}

Returns True iff `self` is compatible with `other`.

Two possibly-partially-defined shapes are compatible if there
exists a fully-defined shape that both shapes can represent. Thus,
compatibility allows the shape inference code to reason about
partially-defined shapes. For example:

* TensorShape(None) is compatible with all shapes.

* TensorShape([None, None]) is compatible with all two-dimensional
  shapes, such as TensorShape([32, 784]), and also TensorShape(None). It is
  not compatible with, for example, TensorShape([None]) or
  TensorShape([None, None, None]).

* TensorShape([32, None]) is compatible with all two-dimensional shapes
  with size 32 in the 0th dimension, and also TensorShape([None, None])
  and TensorShape(None). It is not compatible with, for example,
  TensorShape([32]), TensorShape([32, None, 1]) or TensorShape([64, None]).

* TensorShape([32, 784]) is compatible with itself, and also
  TensorShape([32, None]), TensorShape([None, 784]), TensorShape([None,
  None]) and TensorShape(None). It is not compatible with, for example,
  TensorShape([32, 1, 784]) or TensorShape([None]).

The compatibility relation is reflexive and symmetric, but not
transitive. For example, TensorShape([32, 784]) is compatible with
TensorShape(None), and TensorShape(None) is compatible with
TensorShape([4, 4]), but TensorShape([32, 784]) is not compatible with
TensorShape([4, 4]).

##### Args:


*  <b>`other`</b>: Another TensorShape.

##### Returns:

  True iff `self` is compatible with `other`.


- - -

#### `tf.TensorShape.is_fully_defined()` {#TensorShape.is_fully_defined}

Returns True iff `self` is fully defined in every dimension.



- - -

#### `tf.TensorShape.with_rank(rank)` {#TensorShape.with_rank}

Returns a shape based on `self` with the given rank.

This method promotes a completely unknown shape to one with a
known rank.

##### Args:


*  <b>`rank`</b>: An integer.

##### Returns:

  A shape that is at least as specific as `self` with the given rank.

##### Raises:


*  <b>`ValueError`</b>: If `self` does not represent a shape with the given `rank`.


- - -

#### `tf.TensorShape.with_rank_at_least(rank)` {#TensorShape.with_rank_at_least}

Returns a shape based on `self` with at least the given rank.

##### Args:


*  <b>`rank`</b>: An integer.

##### Returns:

  A shape that is at least as specific as `self` with at least the given
  rank.

##### Raises:


*  <b>`ValueError`</b>: If `self` does not represent a shape with at least the given
    `rank`.


- - -

#### `tf.TensorShape.with_rank_at_most(rank)` {#TensorShape.with_rank_at_most}

Returns a shape based on `self` with at most the given rank.

##### Args:


*  <b>`rank`</b>: An integer.

##### Returns:

  A shape that is at least as specific as `self` with at most the given
  rank.

##### Raises:


*  <b>`ValueError`</b>: If `self` does not represent a shape with at most the given
    `rank`.



- - -

#### `tf.TensorShape.assert_has_rank(rank)` {#TensorShape.assert_has_rank}

Raises an exception if `self` is not compatible with the given `rank`.

##### Args:


*  <b>`rank`</b>: An integer.

##### Raises:


*  <b>`ValueError`</b>: If `self` does not represent a shape with the given `rank`.


- - -

#### `tf.TensorShape.assert_same_rank(other)` {#TensorShape.assert_same_rank}

Raises an exception if `self` and `other` do not have compatible ranks.

##### Args:


*  <b>`other`</b>: Another `TensorShape`.

##### Raises:


*  <b>`ValueError`</b>: If `self` and `other` do not represent shapes with the
    same rank.


- - -

#### `tf.TensorShape.assert_is_compatible_with(other)` {#TensorShape.assert_is_compatible_with}

Raises exception if `self` and `other` do not represent the same shape.

This method can be used to assert that there exists a shape that both
`self` and `other` represent.

##### Args:


*  <b>`other`</b>: Another TensorShape.

##### Raises:


*  <b>`ValueError`</b>: If `self` and `other` do not represent the same shape.


- - -

#### `tf.TensorShape.assert_is_fully_defined()` {#TensorShape.assert_is_fully_defined}

Raises an exception if `self` is not fully defined in every dimension.

##### Raises:


*  <b>`ValueError`</b>: If `self` does not have a known value for every dimension.



#### Other Methods
- - -

#### `tf.TensorShape.__init__(dims)` {#TensorShape.__init__}

Creates a new TensorShape with the given dimensions.

##### Args:


*  <b>`dims`</b>: A list of Dimensions, or None if the shape is unspecified.
*  <b>`DEPRECATED`</b>: A single integer is treated as a singleton list.


- - -

#### `tf.TensorShape.as_dimension_list()` {#TensorShape.as_dimension_list}

DEPRECATED: use as_list().


- - -

#### `tf.TensorShape.num_elements()` {#TensorShape.num_elements}

Returns the total number of elements, or none for incomplete shapes.



- - -

### `class tf.Dimension` {#Dimension}

Represents the value of one dimension in a TensorShape.
- - -

#### `tf.Dimension.__init__(value)` {#Dimension.__init__}

Creates a new Dimension with the given value.


- - -

#### `tf.Dimension.assert_is_compatible_with(other)` {#Dimension.assert_is_compatible_with}

Raises an exception if `other` is not compatible with this Dimension.

##### Args:


*  <b>`other`</b>: Another Dimension.

##### Raises:


*  <b>`ValueError`</b>: If `self` and `other` are not compatible (see
    is_compatible_with).


- - -

#### `tf.Dimension.is_compatible_with(other)` {#Dimension.is_compatible_with}

Returns true if `other` is compatible with this Dimension.

Two known Dimensions are compatible if they have the same value.
An unknown Dimension is compatible with all other Dimensions.

##### Args:


*  <b>`other`</b>: Another Dimension.

##### Returns:

  True if this Dimension and `other` are compatible.


- - -

#### `tf.Dimension.merge_with(other)` {#Dimension.merge_with}

Returns a Dimension that combines the information in `self` and `other`.

Dimensions are combined as follows:

    Dimension(n)   .merge_with(Dimension(n))    == Dimension(n)
    Dimension(n)   .merge_with(Dimension(None)) == Dimension(n)
    Dimension(None).merge_with(Dimension(n))    == Dimension(n)
    Dimension(None).merge_with(Dimension(None)) == Dimension(None)
    Dimension(n)   .merge_with(Dimension(m)) raises ValueError for n != m

##### Args:


*  <b>`other`</b>: Another Dimension.

##### Returns:

  A Dimension containing the combined information of `self` and
  `other`.

##### Raises:


*  <b>`ValueError`</b>: If `self` and `other` are not compatible (see
    is_compatible_with).


- - -

#### `tf.Dimension.value` {#Dimension.value}

The value of this dimension, or None if it is unknown.


- - -

### `tf.op_scope(values, name, default_name)` {#op_scope}

Returns a context manager for use when defining a Python op.

This context manager validates that the given `values` are from the
same graph, ensures that that graph is the default graph, and pushes a
name scope.

For example, to define a new Python op called `my_op`:

```python
def my_op(a, b, c, name=None):
  with tf.op_scope([a, b, c], name, "MyOp") as scope:
    a = tf.convert_to_tensor(a, name="a")
    b = tf.convert_to_tensor(b, name="b")
    c = tf.convert_to_tensor(c, name="c")
    # Define some computation that uses `a`, `b`, and `c`.
    return foo_op(..., name=scope)
```

##### Args:


*  <b>`values`</b>: The list of `Tensor` arguments that are passed to the op function.
*  <b>`name`</b>: The name argument that is passed to the op function.
*  <b>`default_name`</b>: The default name to use if the `name` argument is `None`.

##### Returns:

  A context manager for use in defining a Python op.


- - -

### `tf.get_seed(op_seed)` {#get_seed}

Returns the local seeds an operation should use given an op-specific seed.

Given operation-specific seed, `op_seed`, this helper function returns two
seeds derived from graph-level and op-level seeds. Many random operations
internally use the two seeds to allow user to change the seed globally for a
graph, or for only specific operations.

For details on how the graph-level seed interacts with op seeds, see
[`set_random_seed`](../../api_docs/python/constant_op.md#set_random_seed).

##### Args:


*  <b>`op_seed`</b>: integer.

##### Returns:

  A tuple of two integers that should be used for the local seed of this
  operation.


