User-Defined Stream-Generating Functions
========================================

This section describes how to write a UDSF in Go.

Implementing a UDSF
-------------------

To provide a UDSF, two interfaces need to be provided: ``UDSF`` and
``UDSFCreator``.

The interface ``UDSF`` is defined as follows in
``gopkg.in/sensorbee/sensorbee.v0/bql/udf`` package.

::

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
method is called everytime a new tuple arrives. The argument ``t`` contains the
tuple emitted from another stream. Stream-like UDSFs have to return from
``Process`` immediately after it finished processing tuples. They must not
block in the method. A stream-like UDSF is used mostly when multiple tuples
need to be computed and emitted based on one input tuple::

    type WordSplitter struct {
        Field string
    }

    func (w *WordSplitter) Process(ctx *core.Context, t *core.Tuple, writer core.Writer) error {
        var kwd []string
        if v, ok := t.Data[w.Field]; !ok {
            return fmt.Errorf("the tuple doesn't have the required field: %v", w.Field)
        } else if s, err := data.AsString(v); err != nil {
            return fmt.Errorf("'%v' field must be string: %v", w.Field, err)
        } else {
            kwd = strings.Split(s, " ")
        }

        for _, k := range kwd {
            out := t.Copy()
            out.Data[w.Field] = data.String(k)
            if err := writer.Write(out); err != nil {
                return err
            }
        }
        return nil
    }

    func (w *WordSplitter) Terminate(ctx *core.Context) error {
        return nil
    }

``WordSplitter`` splits text in a specific field by space. For example, when
an input tuple is ``{'word': 'a b c'}`` and ``WordSplitter.Field`` is ``word``,
following three tuples will be emitted: ``{'word': 'a'}``, ``{'word': 'b'}``,
and ``{'word': 'c'}``.

When a UDSF is a source-like UDSF, the ``Process`` method is only called once
with a tuple that doesn't mean anything. Unlike a stream-like UDSF, the
``Process`` method of a source-like UDSF doesn't have to return until it emits
all tuples, the ``Terminate`` method is called, or a fatal error occurs.

::

    type Ticker struct {
        stopped int32
        Interval float64
    }

    func (t *Ticker) Process(ctx *core.Context, t *core.Tuple, w core.Writer) error {
        var i int64
        for ; atomic.LoadInt32(&t.stopped) == 0; i++ {
            if err := w.Write(core.NewTuple(data.Map{"tick": data.Int(i)})); err != nil {
                return err
            }
            time.Sleep(t.Interval)
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
in ``gopkg.in/sensorbee/sensorbee.v0/bql/udf`` package::

    type UDSFCreator interface {
        CreateUDSF(ctx *core.Context, decl UDSFDeclarer, args ...data.Value) (UDSF, error)
        Accept(arity int) bool
    }

    func (w *WordSplitter) Terminate(ctx *core.Context) error {
        return nil
    }

``UDSFCreator`` interface creates a new instance of a UDSF. The ``CreateUDSF``
method creates a new instance of a UDSF. The method is called when evaluating
a UDSF in the ``FROM`` clause of the ``SELECT`` statement. ``ctx`` contains
the processing context information. ``decl`` is used to customize the behavior
of the UDSF, which is explained later. ``args`` has arguments passed in the
``SELECT`` statement. The ``Accept`` method verifies if the UDSF accept the
specific number of arguments. This is same as ``UDF.Arity`` method (see
:ref:`server_programming_go_udsf`).

``UDSFDeclarer`` is used in the ``CreateUDSF`` method to customized the
behavior of a UDSF::

    type UDSFDeclarer interface {
        Input(name string, config *UDSFInputConfig) error
        ListInputs() map[string]*UDSFInputConfig
    }

By calling its ``Input`` method, a UDSF will be able to receive tuples from
another stream having the ``name``. Because the ``name`` is given outside the
UDSF, it's uncontrollable from the UDSF. However, there're cases that a UDSF
wants to know from which stream a tuple has come. For example, when providing
a UDSF performing a JOIN or two streams, a UDSF needs to distinguish which
stream emittted the tuple. If the UDSF was defined as
``my_join(left_stream, right_stream)``, ``decl`` can be used as follows in
``UDSFCreator.CreateUDSF``::

    decl.Input(args[0], &UDSFInputConfig{InputName: "left"})
    decl.Input(args[1], &UDSFInputConfig{InputName: "right"})

By configuring input stream in this way, a tuple passed to ``UDSF.Process`` has
the given name in its ``Tuple.InputName`` field::

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

As an example, the ``UDSFCreator`` of ``WordSpliter`` is shown below::

    type WordSplitterCreator struct {
    }

    func (w *WordSplitterCreator) CreateUDSF(ctx *core.Context,
        decl udf.UDSFDeclarer, args ...data.Value) (udf.UDSF, error) {
        input, err := data.AsString(args[0])
        if err != nil {
            return fmt.Errorf("input stream name must be a string: %v", args[0])
        }
        field, err := data.AsString(args[1])
        if err != nil {
            return fmt.Errorf("target field name must be a string: %v", args[1])
        }
        // This Input call makes the UDSF a stream-like UDSF.
        if err := decl.Input(input); err != nil {
            return err
        }
        return &WordSplitter{
            Field: field,
        }
    }

    func (w *WordSplitterCreator) Accept(arity int) bool {
        return arity == 2
    }

Although the UDSF hasn't been registered to the SensorBee server yet, it could
appear like ``word_splitter(input_stream_name, target_field_name)`` if it's
registered with the name ``word_splitter``.

For another example, the ``UDSFCreator`` of ``Ticker`` is shown below::

    type TickerCreator struct {
    }

    func (t *TickerCreator) CreateUDSF(ctx *core.Context,
        decl udf.UDSFDeclarer, args ...data.Value) (udf.UDSF, error) {
        interval, err := data.ToFloat(args[0])
        // Since this is a source-like UDSF, there's no input.
        return &Ticker{
            Interval: interval,
        }
    }

    func (t *TickerCreator) Accept(arity int) bool {
        return arity == 1
    }

Like ``word_splitter``, its signature could be ``ticker(interval)`` if the UDSF
is registered as ``ticker``.

The implementation of a UDSF is completed and the next step is to register it
to the SensorBee server.

Registering a UDSF
------------------

A UDSF can be used in BQL by registering its ``UDSFCreator`` interface to
the SensorBee server by ``RegisterGlobalUDSFCreator`` or
``MustRegisterGlobalUDSFCreator`` function, which is defined in
``gopkg.in/sensorbee/sensorbee.v0/bql/udf``.

The following example registers ``WordSplitter`` and ``Ticker``::

    func init() {
        udf.RegisterGlobalUDSFCreator("word_splitter", &WordSplitterCreator{})
        udf.RegisterGlobalUDSFCreator("ticker", &TickerCreator{})
    }

Generic UDSFs
-------------

Like UDFs have ``ConvertGeneric`` function, UDSFs also have
``ConvertToUDSFCreator`` and ``MustConvertToUDSFCreator`` function. They convert
a regular function satisfying some restrictions to the ``UDSFCreator`` interface.

The restrictions are the same as
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
function as follows::

    func WordSplitterCreator(decl udf.UDSFDeclarer,
        inputStream, field string) (udf.UDSF, error) {
        if err := decl.Input(inputStream); err != nil {
            return err
        }
        return &WordSplitter{
            Field: field,
        }
    }

    func init() {
        udf.RegisterGlobalUDSFCreator("word_splitter",
            udf.MustConvertToUDSFCreator(WordSplitterCreator))
    }

``TickerCreator`` can be replaced with ``ConvertToUDSFCreator``, too::

    func TickerCreator(decl udf.UDSFDeclarer, interval float64) (udf.UDSF, error) {
        return &Ticker{
            Interval: interval,
        }
    }

    func init() {
        udf.RegisterGlobalUDSFCreator("ticker",
            udf.MustConvertToUDSFCreator(TickerCreator))
    }

A Complete Example
------------------

TODO