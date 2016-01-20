User-Defined Functions
======================

This section describes how to write a UDF in Go. It first shows the basic
interface of defining UDFs, and then describes utilities among it.

Overview
--------

A struct satisfying the following interface can be a UDF::

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

``Call`` method receives ``*core.Context`` and multiple ``data.Value`` as its
arguments. ``*core.Context`` contains the information of the current processing
context. ``data.Value`` represents a value used in BQL and can be any of
:ref:`built-in types <bql_types_types>`.

``Accept`` method verifies if the UDF accept the specific number of arguments.
It can return true for multiple arities as long as it can receive the given
number of arguments.

``IsAggregationParameter`` checks if k-th argument is an aggregation parameter.

A UDF can be registered by ``RegisterGlobalUDF`` or ``MustRegisterGlobalUDF``
function defined in ``gopkg.in/sensorbee/sensorbee.v0/bql/udf`` package.
``MustRegisterGlobalUDF`` panics on failure instead of returning an error.
These functions are usually called from ``init`` function::

    type MyUDF struct {
        ...
    }

    ... implementation of required methods ...

    func init() {
        udf.MustRegisterGlobalUDF("my_udf", &MyUDF{})
    }

As it can be inferred from this example, a UDF itself should be stateless since
it only registers one instance of a struct as a UDF and it'll globally be shared.
Stateful data processing can be achieved by the combination of UDFs and UDSs,
which is described in the following sections.

The registered UDF is looked up based on its name and the number of argument
passed to it.

::

    SELECT RSTREAM my_udf(arg1, arg2) FROM stream [RANGE 1 TUPLES];

In this ``SELECT``, a UDF having the name ``my_udf`` is looked up first. After
that, its ``Accept``method  is called with 2 and ``my_udf`` is actually selected
if ``Accept(2)`` returned true. ``IsAggregationParameter`` method is
additionally called on each argument to see if the argument needs to be an
aggregation parameter. Then, if there's no mismatch, ``my_udf`` is finally
called.

.. note::

    A UDF doesn't have a schema at the moment, so any error regarding types of
    arguments will not be reported until the statement calling the UDF actually
    processes a tuple.

Generic UDFs
------------

SensorBee provides a helper function to register a regular Go function as a UDF.

.. todo:: describe generic UDFs written in wiki

Developing a UDF
----------------

The basic development flow of a UDF is as follows:

#. Create a git repository for a UDF
#. Implement the UDF
#. Create a plugin subpackage in the repository

Create a Git Repository for a UDF
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The UDF is written in Go, so it needs to be a valid git repository (or a
repository of other version control systems). One repository may provide
multiple UDFs. However, since Go is very well designed to provide packages in
a fine-grained manner, each repository should only provide a minimum set of
UDFs that are logically related and make sense to be in the same repository.

Implement the UDF
^^^^^^^^^^^^^^^^^

The next step is to implement the UDF. There's no restiction on which packages
to use.

Functions or structs that are registered to the SensorBee server needs to be
referred by the plugin subpackage, which is described in the next subsection.
Thus, names of those symbols need to start with a capital letter.

In this step, the UDF shouldn't be registered to the SensorBee server yet.

Create a Plugin Subpackage in the Repository
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

It is highly recommended that the repository have a separate package which only
registers UDFs to the SensorBee server. There's usually one file named "plugin.go"
and it only contains a series of ``RegisterGlobalUDF`` calls in ``init``
function. For instance, if the repository only provides one UDF, the contents of
"plugin.go" would be something like::

    // in github.com/user/myudf/plugin/plugin.go
    package plugin

    import (
        "gopkg.in/sensorbee/sensorbee.v0/bql/udf"
        "github.com/user/myudf"
    )

    func init() {
        udf.MustRegisterGlobalUDF("my_udf", &myudf.MyUDF{})
    }

There're two reasons to have a plugin subpackage separated from the
implementation of UDFs. Firstly, by separating them, other Go packages can
import the UDFs implementation to use the package as a library without
registering them to SensorBee. Secondly, having a separated plugin package
allows a user to register a UDF with a different name. This is especially useful
when names of UDFs conflict each other.

To use the example plugin above, "github.com/user/myudf/plugin" needs to be
added to the plugin path list of SensorBee.

Repository Organization
^^^^^^^^^^^^^^^^^^^^^^^

The typical organization of the repository is

* github.com/user/repo

    * README: description and the usage of the UDF
    * .go files: implementation of the UDF
    * plugin/: a subpackage for the plugin registration

        * plugin.go

    * othersubpackages/: there can be optional subpackages

An Example
----------

.. todo:: twice


Dynamic Loading
---------------

Dynamic loading of UDFs written in Go isn't supported at the moment because
Go doesn't officially support loading packages dynamically.
