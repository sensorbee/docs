Sink Plugins
============

This section describes how to implement a sink as a plugin of SensorBee.

Implementing a Sink
-------------------

A struct implementing the following interface can be a sink::

    type Sink interface {
        Write(ctx *Context, t *Tuple) error
        Close(ctx *Context) error
    }

This interface is defined in ``gopkg.in/sensorbee/sensorbee.v0/core`` package.

The ``Write`` method write a tuple to a destination of the sink. The argument
``ctx`` contains the information of the current processing context. ``t`` is the
tuple to be written. The ``Close`` method is called when the sink becomes
unnecessary. It must release all resources allocated for the sink.

The following example sink write a tuple as a JSON to stdout::

    type StdoutSink struct {
    }

    func (s *StdoutSink) Write(ctx *core.Context, t *core.Tuple) error {
        _, err := fmt.Fprintln(os.Stdout, t.Data)
        return err
    }

    func (s *StdoutSink) Close(ctx *core.Context) error {
        // nothing to release
        return nil
    }

A sink is initialized by its ``SinkCreator``, which is described later.

.. note::

    SensorBee doesn't provide buffering or retry capability for sinks.

Registering a Sink
------------------

To register a sink to the SensorBee server, the sink needs to provide its
``SinkCreator``. The ``SinkCreator`` interface is defined in
``gopkg.in/sensorbee/sensorbee.v0/bql`` pacakge as follows::

    // SinkCreator is an interface which creates instances of a Sink.
    type SinkCreator interface {
        // CreateSink creates a new Sink instance using given parameters.
        CreateSink(ctx *core.Context, ioParams *IOParams, params data.Map) (core.Sink, error)
    }

It only has one method: ``CreateSink``. The ``CreateSink`` method is called
when the :ref:`CREATE SINK <ref_stmts_create_sink>` statement is executed.
The ``ctx`` argument contains the information of the current processing context.
``ioParams`` has the name and the type name of the sink, which are given in
the ``CREATE SINK`` statement. ``params`` has parameters specified in the
``WITH`` clause of the ``CREATE SINK`` statement.

The creator can be registered by ``RegisterGlobalSinkCreator`` or
``MustRegisterGlobalSinkCreator`` function. As an example, the creator of
``StdoutSink`` above can be implemented and registered as follows::

    type StdoutSinkCreator struct {
    }

    func (s *StdoutSinkCreator) CreateSink(ctx *core.Context,
        ioParams *bql.IOParams, params data.Map) (core.Sink, error) {
        return &StdoutSink{}, nil
    }

    func init() {
        bql.MustRegisterGlobalSinkCreator("my_stdout", &StdoutSinkCreator{})
    }

This sink doesn't have parameters specified in the ``WITH`` clause of the
``CREATE SINK`` statement. How to handle parameters for sink is same as how
source does. See :ref:`server_programming_go_sources` for more details.

Utilities
---------

There's one utility function for sink plugins: ``SinkCreatorFunc``::

    func SinkCreatorFunc(f func(*core.Context,
        *IOParams, data.Map) (core.Sink, error)) SinkCreator

This utility function is defined in ``gopkg.in/sensorbee/sensorbee.v0/bql``
pacakge. It converts a function having the same signature as
``SinkCreator.CreateSink`` to a ``SinkCreator``. With this utility, for example,
``StdoutSinkCreator`` can be modified to::

    func CreateStdoutSink(ctx *core.Context,
        ioParams *bql.IOParams, params data.Map) (core.Sink, error) {
        return &StdoutSink{}, nil
    }

    fucn init() {
        bql.MustRegisterGlobalSinkCreator("stdout",
            bql.SinkCreatorFunc(CreateStdoutSink))
    }

A Complete Example
------------------

A complete example of the sink is shown in this subsection. The package name for
the sink is ``stdout`` and ``StdoutSink`` is renamed to ``Sink``. Also, this
example uses ``SinkCreatorFunc`` utility for ``SinkCreator``.

Assume that the import path of the example is
``github.com/sensorbee/examples/stdout``, which doesn't actually exist. The
repository has to files:

* stdout.go
* plugin/plugin.go

stdout.go
^^^^^^^^^

::

    package stdout

    import (
        "fmt"
        "os"

        "gopkg.in/sensorbee/sensorbee.v0/bql"
        "gopkg.in/sensorbee/sensorbee.v0/core"
        "gopkg.in/sensorbee/sensorbee.v0/data"
    )

    type Sink struct {
    }

    func (s *Sink) Write(ctx *core.Context, t *core.Tuple) error {
        _, err := fmt.Fprintln(os.Stdout, t.Data)
        return err
    }

    func (s *Sink) Close(ctx *core.Context) error {
        return nil
    }

    func Create(ctx *core.Context, ioParams *bql.IOParams, params data.Map) (core.Sink, error) {
        return &Sink{}, nil
    }


plugin/plugin.go
^^^^^^^^^^^^^^^^

::

    package plugin

    import (
        "gopkg.in/sensorbee/sensorbee.v0/bql"

        "github.com/sensorbee/examples/stdout"
    )

    func init() {
        bql.MustRegisterGlobalSinkCreator("my_stdout",
            bql.SinkCreatorFunc(stdout.Create))
    }
