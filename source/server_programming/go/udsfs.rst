.. _server_programming_go_udsfs:

User-Defined Stream-Generating Functions
========================================

This section describes how to write a UDSF in Go.

Implementing a UDSF
-------------------

To provide a UDSF, two interfaces need to be implemented: ``UDSF`` and
``UDSFCreator``.

The interface ``UDSF`` is defined as follows in the
``gopkg.in/sensorbee/sensorbee.v0/bql/udf`` package.

.. code-block:: go

    type UDSF interface {
        Process(ctx *core.Context, t *core.Tuple, w core.Writer) error
        Terminate(ctx *core.Context) error
    }

The ``Process`` method processes an input tuple and emits computed tuples for
subsequent streams. ``ctx`` contains the processing context information. ``t``
is the tuple to be processed in the UDSF. ``w`` is the destination to where
computed tuples are emitted. The ``Terminate`` method is called when the UDFS
becomes unnecessary. The method has to release all the resources the UDSF has.

How the ``Process`` method is called depends on the type of a UDSF. When a UDFS
is a stream-like UDSF (i.e. it has input from other streams), the ``Process``
method is called every time a new tuple arrives. The argument ``t`` contains the
tuple emitted from another stream. Stream-like UDSFs have to return from
``Process`` immediately after processing the input tuple. They must not
block in the method.

A stream-like UDSF is used mostly when multiple tuples
need to be computed and emitted based on one input tuple:

.. code-block:: go

    type WordSplitter struct {
        field string
    }

    func (w *WordSplitter) Process(ctx *core.Context, t *core.Tuple, writer core.Writer) error {
        var kwd []string
        if v, ok := t.Data[w.field]; !ok {
            return fmt.Errorf("the tuple doesn't have the required field: %v", w.field)
        } else if s, err := data.AsString(v); err != nil {
            return fmt.Errorf("'%v' field must be string: %v", w.field, err)
        } else {
            kwd = strings.Split(s, " ")
        }

        for _, k := range kwd {
            out := t.Copy()
            out.Data[w.field] = data.String(k)
            if err := writer.Write(ctx, out); err != nil {
                return err
            }
        }
        return nil
    }

    func (w *WordSplitter) Terminate(ctx *core.Context) error {
        return nil
    }

``WordSplitter`` splits text in a specific field by space. For example, when
an input tuple is ``{"word": "a b c"}`` and ``WordSplitter.Field`` is ``word``,
following three tuples will be emitted: ``{"word": "a"}``, ``{"word": "b"}``,
and ``{"word": "c"}``.

When a UDSF is a source-like UDSF, the ``Process`` method is only called once
with a tuple that does not mean anything. Unlike a stream-like UDSF, the
``Process`` method of a source-like UDSF does not have to return until it has
emitted all tuples, the ``Terminate`` method is called, or a fatal error occurs.

.. code-block:: go

    type Ticker struct {
        interval time.Duration
        stopped  int32
    }

    func (t *Ticker) Process(ctx *core.Context, tuple *core.Tuple, w core.Writer) error {
        var i int64
        for ; atomic.LoadInt32(&t.stopped) == 0; i++ {
            newTuple := core.NewTuple(data.Map{"tick": data.Int(i)})
            if err := w.Write(ctx, newTuple); err != nil {
                return err
            }
            time.Sleep(t.interval)
        }
        return nil
    }

    func (t *Ticker) Terminate(ctx *core.Context) error {
        atomic.StoreInt32(&t.stopped, 1)
        return nil
    }

In this example, ``Ticker`` emits tuples having ``tick`` field containing
a counter until the ``Terminate`` method is called.

Whether a UDSF is stream-like or source-like can be configured when it is
created by ``UDSFCreator``. The interface ``UDSFCreator`` is defined as follows
in ``gopkg.in/sensorbee/sensorbee.v0/bql/udf`` package:

.. code-block:: go

    type UDSFCreator interface {
        CreateUDSF(ctx *core.Context, decl UDSFDeclarer, args ...data.Value) (UDSF, error)
        Accept(arity int) bool
    }

The ``CreateUDSF`` method creates a new instance of a UDSF. The method is called
when evaluating a UDSF in the ``FROM`` clause of a ``SELECT`` statement. ``ctx``
contains the processing context information. ``decl`` is used to customize the
behavior of the UDSF, which is explained later. ``args`` has arguments passed in the
``SELECT`` statement. The ``Accept`` method verifies if the UDSF accept the
specific number of arguments. This is the same as ``UDF.Arity`` method (see
:ref:`server_programming_go_udfs`).

``UDSFDeclarer`` is used in the ``CreateUDSF`` method to customize the
behavior of a UDSF:

.. code-block:: go

    type UDSFDeclarer interface {
        Input(name string, config *UDSFInputConfig) error
        ListInputs() map[string]*UDSFInputConfig
    }

By calling its ``Input`` method, a UDSF will be able to receive tuples from
another stream with the name ``name``. Because the ``name`` is given outside the
UDSF, it's uncontrollable from the UDSF. However, there are cases that a UDSF
wants to know from which stream a tuple has come. For example, when providing
a UDSF performing a JOIN or two streams, a UDSF needs to distinguish which
stream emitted the tuple. If the UDSF was defined as
``my_join(left_stream, right_stream)``, ``decl`` can be used as follows in
``UDSFCreator.CreateUDSF``:

.. code-block:: go

    decl.Input(args[0], &UDSFInputConfig{InputName: "left"})
    decl.Input(args[1], &UDSFInputConfig{InputName: "right"})

By configuring the input stream in this way, a tuple passed to ``UDSF.Process`` has
the given name in its ``Tuple.InputName`` field:

.. code-block:: go

    func (m *MyJoin) Process(ctx *core.Context, t *core.Tuple, w core.Writer) error {
        switch t.InputName {
        case "left":
            ... process tuples from left_stream ...
        case "right":
            ... process tuples from right_stream ...
        }
        ...
    }

If a UDSF is configured to have one or more input streams by ``decl.Input`` in
the ``UDSFCreator.CreateUDSF`` method, the UDSF is processed as a stream-like
UDSF. Otherwise, if a UDSF doesn't have any input (i.e. ``decl.Input`` is not
called), the UDSF becomes a source-like UDSF.

As an example, the ``UDSFCreator`` of ``WordSplitter`` is shown below:

.. code-block:: go

    type WordSplitterCreator struct {
    }

    func (w *WordSplitterCreator) CreateUDSF(ctx *core.Context,
        decl udf.UDSFDeclarer, args ...data.Value) (udf.UDSF, error) {
        input, err := data.AsString(args[0])
        if err != nil {
            return nil, fmt.Errorf("input stream name must be a string: %v", args[0])
        }
        field, err := data.AsString(args[1])
        if err != nil {
            return nil, fmt.Errorf("target field name must be a string: %v", args[1])
        }
        // This Input call makes the UDSF a stream-like UDSF.
        if err := decl.Input(input, nil); err != nil {
            return nil, err
        }
        return &WordSplitter{
            field: field,
        }, nil
    }

    func (w *WordSplitterCreator) Accept(arity int) bool {
        return arity == 2
    }

Although the UDSF has not been registered to the SensorBee server yet, it could
appear like ``word_splitter(input_stream_name, target_field_name)`` if it was
registered with the name ``word_splitter``.

For another example, the ``UDSFCreator`` of ``Ticker`` is shown below:

.. code-block:: go

    type TickerCreator struct {
    }

    func (t *TickerCreator) CreateUDSF(ctx *core.Context,
        decl udf.UDSFDeclarer, args ...data.Value) (udf.UDSF, error) {
        interval, err := data.ToDuration(args[0])
        if err != nil {
            return nil, err
        }
        // Since this is a source-like UDSF, there's no input.
        return &Ticker{
            interval: interval,
        }, nil
    }

    func (t *TickerCreator) Accept(arity int) bool {
        return arity == 1
    }

Like ``word_splitter``, its signature could be ``ticker(interval)`` if the UDSF
is registered as ``ticker``.

The implementation of this UDSF is completed and the next step is to register it
to the SensorBee server.

Registering a UDSF
------------------

A UDSF can be used in BQL by registering its ``UDSFCreator`` interface to
the SensorBee server using the ``RegisterGlobalUDSFCreator`` or
``MustRegisterGlobalUDSFCreator`` functions, which are defined in
``gopkg.in/sensorbee/sensorbee.v0/bql/udf``.

The following example registers ``WordSplitter`` and ``Ticker``:

.. code-block:: go

    func init() {
        udf.RegisterGlobalUDSFCreator("word_splitter", &WordSplitterCreator{})
        udf.RegisterGlobalUDSFCreator("ticker", &TickerCreator{})
    }

Generic UDSFs
-------------

Just like UDFs have a ``ConvertGeneric`` function, UDSFs also have
``ConvertToUDSFCreator`` and ``MustConvertToUDSFCreator`` function. They convert
a regular function satisfying some restrictions to the ``UDSFCreator`` interface.

The restrictions are the same as for
:ref:`generic UDFs <server_programming_go_udfs_generic_udfs>` except that a
function converted to the ``UDSFCreator`` interface has an additional argument
``UDSFDeclarer``. ``UDSFDeclarer`` is located after ``*core.Context`` and before
other arguments. Examples of valid function signatures are show below:

* ``func(*core.Context, UDSFDeclarer, int)``
* ``func(UDSFDeclarer, string)``
* ``func(UDSFDeclarer)``
* ``func(*core.Context, UDSFDeclarer, ...data.Value)``
* ``func(UDSFDeclarer, ...float64)``
* ``func(*core.Context, UDSFDeclarer, int, ...string)``
* ``func(UDSFDeclarer, int, float64, ...time.Time)``

Unlike ``*core.Context``, ``UDSFDeclarer`` cannot be omitted. The same set of
types can be used for arguments as types that ``ConvertGeneric`` function
accepts.

``WordSplitterCreator`` can be rewritten with the ``ConvertToUDSFCreator``
function as follows:

.. code-block:: go

    func CreateWordSplitter(decl udf.UDSFDeclarer,
        inputStream, field string) (udf.UDSF, error) {
        if err := decl.Input(inputStream, nil); err != nil {
            return nil, err
        }
        return &WordSplitter{
            field: field,
        }, nil
    }

    func init() {
        udf.RegisterGlobalUDSFCreator("word_splitter",
            udf.MustConvertToUDSFCreator(WordSplitterCreator))
    }

``TickerCreator`` can be replaced with ``ConvertToUDSFCreator``, too::

    func CreateTicker(decl udf.UDSFDeclarer, i data.Value) (udf.UDSF, error) {
        interval, err := data.ToDuration(i)
        if err != nil {
            return nil, err
        }
        return &Ticker{
            interval: interval,
        }, nil
    }

    func init() {
        udf.MustRegisterGlobalUDSFCreator("ticker",
           udf.MustConvertToUDSFCreator(udsfs.CreateTicker))
    }

A Complete Example
------------------

This subsection provides a complete example of UDSFs described in this section.
In addition to ``word_splitter`` and ``ticker``, the example also includes the
``lorem`` source, which periodically emits random texts as
``{"text": "lorem ipsum dolor sit amet"}``.

Assume that the import path of the example repository is
``github.com/sensorbee/examples/udsfs``, which doesn't actually exist. The
repository has four files:

* ``lorem.go``
* ``splitter.go``
* ``ticker.go``
* ``plugin/plugin.go``

lorem.go
^^^^^^^^

To learn how to implement a source plugin, see
:ref:`server_programming_go_sources`.

.. code-block:: go

    package udsfs

    import (
        "math/rand"
        "strings"
        "time"

        "gopkg.in/sensorbee/sensorbee.v0/bql"
        "gopkg.in/sensorbee/sensorbee.v0/core"
        "gopkg.in/sensorbee/sensorbee.v0/data"
    )

    var (
        Lorem = strings.Split(strings.Replace(`lorem ipsum dolor sit amet
    consectetur adipiscing elit sed do eiusmod tempor incididunt ut labore et dolore
    magna aliqua Ut enim ad minim veniam quis nostrud exercitation ullamco laboris
    nisi ut aliquip ex ea commodo consequat Duis aute irure dolor in reprehenderit
    in voluptate velit esse cillum dolore eu fugiat nulla pariatur Excepteur sint
    occaecat cupidatat non proident sunt in culpa qui officia deserunt mollit anim
    id est laborum`, "\n", " ", -1), " ")
    )

    type LoremSource struct {
        interval time.Duration
    }

    func (l *LoremSource) GenerateStream(ctx *core.Context, w core.Writer) error {
        for {
            var text []string
            for l := rand.Intn(5) + 5; l > 0; l-- {
                text = append(text, Lorem[rand.Intn(len(Lorem))])
            }

            t := core.NewTuple(data.Map{
                "text": data.String(strings.Join(text, " ")),
            })
            if err := w.Write(ctx, t); err != nil {
                return err
            }

            time.Sleep(l.interval)
        }
    }

    func (l *LoremSource) Stop(ctx *core.Context) error {
        return nil
    }

    func CreateLoremSource(ctx *core.Context,
        ioParams *bql.IOParams, params data.Map) (core.Source, error) {
        interval := 1 * time.Second
        if v, ok := params["interval"]; ok {
            i, err := data.ToDuration(v)
            if err != nil {
                return nil, err
            }
            interval = i
        }
        return core.ImplementSourceStop(&LoremSource{
            interval: interval,
        }), nil
    }

splitter.go
^^^^^^^^^^^

.. code-block:: go

    package udsfs

    import (
        "fmt"
        "strings"

        "gopkg.in/sensorbee/sensorbee.v0/bql/udf"
        "gopkg.in/sensorbee/sensorbee.v0/core"
        "gopkg.in/sensorbee/sensorbee.v0/data"
    )

    type WordSplitter struct {
        field string
    }

    func (w *WordSplitter) Process(ctx *core.Context,
        t *core.Tuple, writer core.Writer) error {
        var kwd []string
        if v, ok := t.Data[w.field]; !ok {
            return fmt.Errorf("the tuple doesn't have the required field: %v", w.field)
        } else if s, err := data.AsString(v); err != nil {
            return fmt.Errorf("'%v' field must be string: %v", w.field, err)
        } else {
            kwd = strings.Split(s, " ")
        }

        for _, k := range kwd {
            out := t.Copy()
            out.Data[w.field] = data.String(k)
            if err := writer.Write(ctx, out); err != nil {
                return err
            }
        }
        return nil
    }

    func (w *WordSplitter) Terminate(ctx *core.Context) error {
        return nil
    }

    func CreateWordSplitter(decl udf.UDSFDeclarer,
        inputStream, field string) (udf.UDSF, error) {
        if err := decl.Input(inputStream, nil); err != nil {
            return nil, err
        }
        return &WordSplitter{
            field: field,
        }, nil
    }

ticker.go
^^^^^^^^^

.. code-block:: go

    package udsfs

    import (
        "sync/atomic"
        "time"

        "gopkg.in/sensorbee/sensorbee.v0/bql/udf"
        "gopkg.in/sensorbee/sensorbee.v0/core"
        "gopkg.in/sensorbee/sensorbee.v0/data"
    )

    type Ticker struct {
        interval time.Duration
        stopped  int32
    }

    func (t *Ticker) Process(ctx *core.Context, tuple *core.Tuple, w core.Writer) error {
        var i int64
        for ; atomic.LoadInt32(&t.stopped) == 0; i++ {
            newTuple := core.NewTuple(data.Map{"tick": data.Int(i)})
            if err := w.Write(ctx, newTuple); err != nil {
                return err
            }
            time.Sleep(t.interval)
        }
        return nil
    }

    func (t *Ticker) Terminate(ctx *core.Context) error {
        atomic.StoreInt32(&t.stopped, 1)
        return nil
    }

    func CreateTicker(decl udf.UDSFDeclarer, i data.Value) (udf.UDSF, error) {
        interval, err := data.ToDuration(i)
        if err != nil {
            return nil, err
        }
        return &Ticker{
            interval: interval,
        }, nil
    }

plugin/plugin.go
^^^^^^^^^^^^^^^^

.. code-block:: go

    package plugin

    import (
        "gopkg.in/sensorbee/sensorbee.v0/bql"
        "gopkg.in/sensorbee/sensorbee.v0/bql/udf"

        "github.com/sensorbee/examples/udsfs"
    )

    func init() {
        bql.MustRegisterGlobalSourceCreator("lorem",
            bql.SourceCreatorFunc(udsfs.CreateLoremSource))
        udf.MustRegisterGlobalUDSFCreator("word_splitter",
            udf.MustConvertToUDSFCreator(udsfs.CreateWordSplitter))
        udf.MustRegisterGlobalUDSFCreator("ticker",
            udf.MustConvertToUDSFCreator(udsfs.CreateTicker))
    }

Example BQL Statements
^^^^^^^^^^^^^^^^^^^^^^

::

    CREATE SOURCE lorem TYPE lorem;
    CREATE STREAM lorem_words AS
        SELECT RSTREAM * FROM word_splitter("lorem", "text") [RANGE 1 TUPLES];

Results of ``word_splitter`` can be received by the following ``SELECT``::

    SELECT RSTREAM * FROM lorem_words [RANGE 1 TUPLES];
