.. _server_programming_go_udfs:

User-Defined Functions
======================

This section describes how to write a UDF in Go. It first shows the basic
interface of defining UDFs, and then describes utilities among it, how to
develop a UDF in a Go manner, and an complete example.

Implementing a UDF
------------------

.. note::

    This is a strict way to implement a UDF in Go. To know a easier way, see
    :ref:`server_programming_go_udfs_generic_udfs`.

A struct implementing the following interface can be a UDF::

    type UDF interface {
        // Call calls the UDF.
        Call(*core.Context, ...data.Value) (data.Value, error)

        // Accept checks if the function accepts the given number of arguments
        // excluding core.Context.
        Accept(arity int) bool

        // IsAggregationParameter returns true if the k-th parameter expects
        // aggregated values. A UDF with Accept(n) == true is an aggregate
        // function if and only if this function returns true for one or more
        // values of k in the range 0, ..., n-1.
        IsAggregationParameter(k int) bool
    }

This interface is defined in ``gopkg.in/sensorbee/sensorbee.v0/bql/udf`` package.

A UDF can be registered by ``RegisterGlobalUDF`` or ``MustRegisterGlobalUDF``
function also defined in ``gopkg.in/sensorbee/sensorbee.v0/bql/udf`` package.
``MustRegisterGlobalUDF`` is same as ``RegisterGlobalUDF`` but panics on failure
instead of returning an error. These functions are usually called from ``init``
function. A typical implementation of a UDF looks as follows::

    type MyUDF struct {
        ...
    }

    func (m *MyUDF) Call(ctx *core.Context, args ...data.Value) (data.Value, error) {
        ...
    }

    func (m *MyUDF) Accept(arity int) bool {
        ...
    }

    func (m *MyUDF) IsAggregationParameter(k int) bool {
        ...
    }

    func init() {
        // MyUDF can be referred as my_udf in BQL statements.
        udf.MustRegisterGlobalUDF("my_udf", &MyUDF{})
    }

As it can be inferred from this example, a UDF itself should be stateless since
it only registers one instance of a struct as a UDF and it'll globally be shared.
Stateful data processing can be achieved by the combination of UDFs and UDSs,
which is described in :ref:`server_programming_go_states`.

A UDF needs to implement three methods to satisfy ``udf.UDF`` interface:
``Call``, ``Accept``, and ``IsAggregationParameter``.

``Call`` method receives ``*core.Context`` and multiple ``data.Value`` as its
arguments. ``*core.Context`` contains the information of the current processing
context. ``Call``'s ``...data.Value`` argument has values passed to the UDF.
``data.Value`` represents a value used in BQL and can be any of :ref:`built-in
types <bql_types_types>`.

::

    SELECT RSTREAM my_udf(arg1, arg2) FROM stream [RANGE 1 TUPLES];

In this example, ``arg1`` and ``arg2`` are passed to ``Call`` method::

    func (m *MyUDF) Call(ctx *core.Context, args ...data.Value) (data.Value, error) {
        // When my_udf(arg1, arg2) is called, len(args) is 2.
        // args[0] is arg1 and args[1] is arg2.
        // It is guaranteed that m.Accept(len(args)) is always true.
    }

Because ``data.Value`` is a semi-variant type, ``Call`` method needs to check
the type of each ``data.Value`` and convert it to a desired type.

``Accept`` method verifies if the UDF accept the specific number of arguments.
It can return true for multiple arities as long as it can receive the given
number of arguments. If a UDF only accept 2 arguments, the method is implemented
as follows::

    func (m *MyUDF) Accept(arity int) bool {
        return arity == 2
    }

When a UDF wants to support variadic parameters (a.k.a. variable-length
arguments) with 2 required arguments (e.g.
``my_udf(arg1, arg2, optional1, optional2, ...)``), the implementation would be::

    func (m *MyUDF) Accept(arity int) bool {
        return arity >= 2
    }

``IsAggregationParameter`` checks if k-th, starting from 0, argument is an
aggregation parameter. Aggregation parameters are passed as ``data.Array``
containing all values of a field in each group.

All of these methods can be called concurrently from multiple goroutines and
they must be thread-safe.

The registered UDF is looked up based on its name and the number of argument
passed to it.

::

    SELECT RSTREAM my_udf(arg1, arg2) FROM stream [RANGE 1 TUPLES];

In this ``SELECT``, a UDF having the name ``my_udf`` is looked up first. After
that, its ``Accept`` method  is called with 2 and ``my_udf`` is actually selected
if ``Accept(2)`` returned true. ``IsAggregationParameter`` method is
additionally called on each argument to see if the argument needs to be an
aggregation parameter. Then, if there's no mismatch, ``my_udf`` is finally
called.

.. note::

    A UDF doesn't have a schema at the moment, so any error regarding types of
    arguments will not be reported until the statement calling the UDF actually
    processes a tuple.

.. _server_programming_go_udfs_generic_udfs:

Generic UDFs
------------

SensorBee provides a helper function to register a regular Go function as a UDF
without implementing ``UDF`` interface explicitly.

::

    func Inc(v int) int {
        return v + 1
    }

This function ``Inc`` can be transformed into a UDF by ``ConvertGeneric``
or ``MustConvertGeneric`` function defined in
``gopkg.in/sensorbee/sensorbee.v0/bql/udf`` package. By combining it with
``RegisterGlobalUDF``, ``Inc`` function can be registered as a UDF::

    func init() {
        udf.MustRegisterGlobalUDF("inc", udf.MustConvertGeneric(Inc))
    }

So, a complete example of the UDF implementation and registration is as follows::

    package inc

    import (
        "gopkg.in/sensorbee/sensorbee.v0/bql/udf"
    )

    func Inc(v int) int {
        return v + 1
    }

    func init() {
        udf.MustRegisterGlobalUDF("inc", udf.MustConvertGeneric(Inc))
    }

.. note::

    A UDF implementation and registration should actually be separated to
    different packages. See :ref:`server_programming_go_development_flow`
    for details.

Although this approach is handy, there could be some overhead compared to a UDF
implemented in the regular way. Most of such overhead comes from type checking
and conversions.

Functions passed to ``ConvertGeneric`` need to satisfy some restrictions on
the form of their argument and return value types. Each restriction is described
in the following subsections.

Form of Arguments
^^^^^^^^^^^^^^^^^

In terms of valid argument forms, there're some rules to follow:

#. A Go function can receive ``*core.Context`` as the first argument, or can omit it.
#. A function can have any number of arguments including 0 argument as long as Go accepts them.
#. A function can be variadic with or without non-variadic parameters.

There're basically eight (four times two, whether a function has
``*core.Context`` or not) forms of arguments (return values are
intentionally omitted for clarity):

* Functions receiving no argument in BQL (e.g. ``my_udf()``)

    1. ``func(*core.Context)``: A function only receiving ``*core.Context``
    2. ``func()``: A function having no argument and not receiving ``*core.Context``, either

* Functions having non-variadic arguments but no variadic arguments

    3. ``func(*core.Context, T1, T2, ..., Tn)``
    4. ``func(T1, T2, ..., Tn)``

* Functions having variadic arguments but no non-variadic arguments

    5. ``func(*core.Context, ...T)``
    6. ``func(...T)``

* Functions having both variadic and non-variadic arguments

    7. ``func(*core.Context, T1, T2, ..., Tn, ...Tn+1)``
    8. ``func(T1, T2, ..., Tn, ...Tn+1)``

Followings are examples of invalid function signatures:

* ``func(T, *core.Context)``: ``*core.Context`` must be the first argument.
* ``func(NonSupportedType)``: Only supported types, which will be explained later, can be used.

Although return values are omitted from all the examples above, they're actually
required. The next subsection explains how to define valid return values.

Form of Return Values
^^^^^^^^^^^^^^^^^^^^^

All functions need to have return values. There're two forms of return values:

* ``func(...) R``
* ``func(...) (R, error)``

All other forms are invalid:

* ``func(...)``
* ``func(...) error``
* ``func(...) NonSupportedType``

Valid types of return values are same as the valid types of arguments, and
they'll be listed in the following subsection.

Valid Value Types
^^^^^^^^^^^^^^^^^

The list of Go types that can be used for arguments and the return value is as
follows:

* ``bool``
* signed integers: ``int``, ``int8``, ``int16``, ``int32``, ``int64``
* unsigned integers: ``uint``, ``uint8``, ``uint16``, ``uint32``, ``uint64``
* ``float32``, ``float64``
* ``string``
* ``time.Time``
* data: ``data.Bool``, ``data.Int``, ``data.Float``, ``data.String``,
  ``data.Blob``, ``data.Timestamp``, ``data.Array``, ``data.Map``, ``data.Value``
* A slice of any type above, including ``data.Value``

``data.Value`` can be used as a semi-variant type, which will receive all types
above.

When the argument type and the actual value type are different, weak type
conversion are applied to values. Conversions are basically done by
``data.ToXXX`` functions (see godoc comments of each function in
data/type_conversions.go). For example, ``func inc(i int) int`` can be called by
``inc("3")`` in a BQL statement and it'll return 4. If a strict type checking
or custom type conversion is required, receive values as ``data.Value`` and
manually check or convert types, or define the UDF in the regular way.

Examples of Valid Go Functions
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Following functions can be converted to UDFs by ``ConvertGeneric`` or
``MustConvertGeneric`` function:

* ``func rand() int``
* ``func pow(*core.Context, float32, float32) (float32, error)``
* ``func join(*core.Context, ...string) string``
* ``func format(string, ...data.Value) (string, error)``
* ``func keys(data.Map) []string``

Complete Examples
-----------------

This subsection shows three example UDFs:

* ``my_inc``
* ``my_join``
* ``my_join2``

Assume that these are in the repository ``github.com/sensorbee/examples/udfs``,
which doesn't actually exist. The repository has three files:

* inc.go
* join.go
* plugin/plugin.go

inc.go
^^^^^^

In inc.go, the ``Inc`` function is defined as a pure Go function with a standard
value type.

::

    package udfs

    func Inc(v int) int {
        return v + 1
    }

join.go
^^^^^^^

In join.go, the ``Join`` UDF is defined in a strict way. It also performs
strict type checking. It's designed to be called with two types of forms:
``my_join("a", "b", "c", "separator")`` or
``my_join(["a", "b", "c"], "separator")``. Each argument and values in the array
must be strings. The UDF receives arbitrary number of arguments.

::

    package udfs

    import (
        "errors"
        "strings"

        "pfi/sensorbee/sensorbee/core"
        "pfi/sensorbee/sensorbee/data"
    )

    type Join struct {
    }

    func (j *Join) Call(ctx *core.Context, args ...data.Value) (data.Value, error) {
        empty := data.String("")
        if len(args) == 1 {
            return empty, nil
        }

        switch args[0].Type() {
        case data.TypeString: // my_join("a", "b", "c", "sep") form
            var ss []string
            for _, v := range args {
                s, err := data.AsString(v)
                if err != nil {
                    return empty, err
                }
                ss = append(ss, s)
            }
            return data.String(strings.Join(ss[:len(ss)-1], ss[len(ss)-1])), nil

        case data.TypeArray: // my_join(["a", "b", "c"], "sep") form
            if len(args) != 2 {
                return empty, errors.New("wrong number of arguments for my_join(array, sep)")
            }
            sep, err := data.AsString(args[1])
            if err != nil {
                return empty, err
            }

            a, _ := data.AsArray(args[0])
            var ss []string
            for _, v := range a {
                s, err := data.AsString(v)
                if err != nil {
                    return empty, err
                }
                ss = append(ss, s)
            }
            return data.String(strings.Join(ss, sep)), nil

        default:
            return empty, errors.New("the first argument must be a string or an array")
        }
    }

    func (j *Join) Accept(arity int) bool {
        return arity >= 1
    }

    func (j *Join) IsAggregationParameter(k int) bool {
        return false
    }

plugin/plugin.go
^^^^^^^^^^^^^^^^

In addition to ``Inc`` and ``Join``, this file registers the standard Go
function ``strings.Join`` as ``my_join2``. Because it's converted to a UDF by
``udf.MustConvertGeneric``, arguments are weakly converted to given types.
For example, ``my_join([1, 2.3, "4"], "-")`` is valid although ``strings.Join``
itself is ``func([]string, string) string``.

::

    package plugin

    import (
        "strings"

        "pfi/sensorbee/sensorbee/bql/udf"

        "pfi/nobu/docexamples/udfs"        
    )

    func init() {
        udf.MustRegisterGlobalUDF("my_inc", udf.MustConvertGeneric(udfs.Inc))
        udf.MustRegisterGlobalUDF("my_join", &udfs.Join{})
        udf.MustRegisterGlobalUDF("my_join2", udf.MustConvertGeneric(strings.Join))
    }

Evaluating Examples
^^^^^^^^^^^^^^^^^^^

Once the ``sensorbee`` command is built with those UDFs and a topology is
created on the server, the ``EVAL`` statement can be used to test them::

    EVAL my_inc(1); -- => 2
    EVAL my_inc(1.5); -- => 2
    EVAL my_inc("10"); -- => 11

    EVAL my_join("a", "b", "c", "-"); -- => "a-b-c"
    EVAL my_join(["a", "b", "c"], ",") -- => "a,b,c"
    EVAL my_join(1, "b", "c", "-") -- => error
    EVAL my_join([1, "b", "c"], ",") -- => error

    EVAL my_join2(["a", "b", "c"], ",") -- => "a,b,c"
    EVAL my_join2([1, "b", "c"], ",") -- => "1,b,c"

Dynamic Loading
---------------

Dynamic loading of UDFs written in Go isn't supported at the moment because
Go doesn't officially support loading packages dynamically.
