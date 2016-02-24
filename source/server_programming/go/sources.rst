.. _server_programming_go_sources:

Source Plugins
==============

This section describes how to implement a source as a plugin of SensorBee.

Implementing a Source
---------------------

A struct implementing the following interface can be a source::

    type Source interface {
        GenerateStream(ctx *Context, w Writer) error
        Stop(ctx *Context) error
    }

This interface is defined in ``gopkg.in/sensorbee/sensorbee.v0/core`` package.

The ``GenerateStream`` methods actually generate tuples for subsequent streams.
The argument ``ctx`` contains the information of the current processing context.
``w`` is the destination to where generated tuples are emitted. The ``Stop``
method stops ``GenerateStream``. It should wait until the ``GenerateStream``
method call returns, but it isn't mandatory.

Once the ``GenerateStream`` method is called, a source can emit as many tuples
as many tuples as it requires. A source basically needs to return from its
``GenerateStream`` method when:

1. it emitted all the tuples it has
2. the ``Stop`` method was called
3. a fatal error occurred

The ``Stop`` method can be called concurrently while the ``GenerateStream``
method is working and it must be thread-safe. As long as a source is used by
components defined in SensorBee, it's guaranteed that its ``Stop`` method is
called only once and it doesn't have to be idempotent. However, it is
recommended that a source provide a termination check in its ``Stop`` method
to avoid a double free problem.

A typical implementation of a source is shown below::

    func (s *MySource) GenerateStream(ctx *core.Context, w core.Writer) error {
        <initialization>
        defer func() {
            <clean up>
        }()

        for <check stop> {
            t := <create a new tuple>
            if err := w.Write(ctx, t); err != nil {
                return err
            }
        }
        return nil
    }

    func (s *MySource) Stop(ctx *core.Context) error {
        <turn on a stop flag>
        <wait until GenerateStream stops>
        return nil
    }

The following example source emits tuple periodically::

    type Ticker struct {
        interval time.Duration
        stopped  int32
    }

    func (t *Ticker) GenerateStream(ctx *core.Context, w core.Writer) error {
        var cnt int64
        for ; ; cnt++ {
            if atomic.LoadInt32(&t.stopped) != 0 {
                break
            }

            tuple := core.NewTuple(data.Map{"tick": data.Int(cnt)})
            if err := w.Write(ctx, tuple); err != nil {
                return err
            }
            time.Sleep(t.interval)
        }
        return nil
    }

    func (t *Ticker) Stop(ctx *core.Context) error {
        atomic.StoreInt32(&t.stopped, 1)
        return nil
    }

The ``interval`` field is initialized in ``SourceCreator``, which is described
later. This is the source version of the example in UDSF's section. This
implementation is a little wrong since the ``Stop`` method doesn't wait until
the ``GenerateStream`` method actually returns. Because implementing a
thread-safe source which stops correctly is a difficult task, ``core`` package
provides a utility function that implements a source's ``Stop`` method on behalf
of the source itself. See :ref:`server_programming_go_sources_utilities` for
details.

Registering a Source
--------------------

To register a source to the SensorBee server, the source needs to provide its
``SourceCreator``. The ``SourceCreator`` interface is defined in
``gopkg.in/sensorbee/sensorbee.v0/bql`` pacakge as follows::

    type SourceCreator interface {
        CreateSource(ctx *core.Context, ioParams *IOParams, params data.Map) (core.Source, error)
    }

It only has one method: ``CreateSource``. The ``CreateSource`` method is called
when the :ref:`CREATE SOURCE <ref_stmts_create_source>` statement is executed.
The ``ctx`` argument contains the information of the current processing context.
``ioParams`` has the name and the type name of the source, which are given in
the ``CREATE SOURCE`` statement. ``params`` has parameters specified in the
``WITH`` clause of the ``CREATE SOURCE`` statement.

The creator can be registered by ``RegisterGlobalSourceCreator`` or
``MustRegisterGlobalSourceCreator`` function. As an example, the creator of
``Ticker`` above can be implemented and registered as follows::

    type TickerCreator struct {
    }

    func (t *TickerCreator) CreateSource(ctx *core.Context,
        ioParams *bql.IOParams, params data.Map) (core.Source, error) {
        interval := 1 * time.Second
        if v, ok := params["interval"]; ok {
            i, err := data.ToDuration(v)
            if err != nil {
                return nil, err
            }
            interval = i
        }
        return &Ticker{
            interval: interval,
        }, nil
    }

    func init() {
        bql.MustRegisterGlobalSourceCreator("ticker", &TickerCreator{})
    }

In this example, the source has a parameter ``interval`` which can be specified
in the ``WITH`` clause of the ``CREATE SOURCE`` statement::

    CREATE SOURCE my_ticker TYPE ticker WITH interval = 0.1;

``my_ticker`` emits tuples that look like ``{"tick": 123}`` in every 100ms.
Without the ``interval`` parameter, ``my_ticker`` will emit tuples in every one
second by default.

Types of a Source
-----------------

In addition to a regular source, there're two more types of sources: a resumable
source and a rewindable source. This subsection describes those sources in
detail.

Resumable Sources
^^^^^^^^^^^^^^^^^

A source that supports :ref:`PAUSE SOURCE <ref_stmts_pause_source>` and
the :ref:`RESUME SOURCE <ref_stmts_resume_source>` statements are called a
resumable source.

Although all sources support them by default, which is done by the ``core``
package, a source can explicitly implement ``core.Resumable`` interface so that
it can provide more efficient pause and resume capability::

    type Resumable interface {
        Pause(ctx *Context) error
        Resume(ctx *Context) error
    }

The ``Pause`` method is called when ``PAUSE SOURCE`` statement is executed and
the ``Resume`` method is called by ``RESUME SOURCE``. The ``Pause`` method may
be called even when the source is already paused, so is the ``Resume`` method.

A source can be non-resumable by implementing these method to return an error::

    type MyNonResumableSource struct {
        ...
    }

    ...

    func (m *MyNonResumableSource) Pause(ctx *core.Context) error {
        return errors.New("my_non_resumable_source doesn't support pause")
    }

    func (m *MyNonResumableSource) Resume(ctx *core.Context) error {
        return errors.New("my_non_resumable_source doesn't support resume")
    }

Rewindable Sources
^^^^^^^^^^^^^^^^^^

A rewindable source can re-generate the same tuples again from the beginning
after it emits all tuples or while it's emitting tuples. A rewindable source
supports the :ref:`REWIND SOURCE <ref_stmts_rewind_source>` statement.

A source can become rewindable by implementing the ``core.RewindableSource``
interface::

    type RewindableSource interface {
        Source
        Resumable

        Rewind(ctx *Context) error
    }

A rewindable source also needs to implement ``core.Resumable`` to be rewindable.

.. note::

    The reason that a rewindable source also needs to be resumable is due to
    the internal implementation of the default pause/resume support. While a
    source is paused, it blocks ``core.Writer.Write`` called in the
    ``GenerateStream`` method. The ``Rewind`` method could also be blocked while
    the ``Write`` call is being blocked until the ``Resume`` method is called.
    It, of course, depends on the implementation of a source, but it's very
    error-prone. Therefore, implementing the ``Resumable`` interface is required
    to be rewindable at the moment.

Unlike a regular source, the ``GenerateStream`` method of a rewindable source
must not return after it emits all tuples. Instead, it needs to wait until the
``Rewind`` method or the ``Stop`` method is called. Once it returns, the source
is considered stopped and no further operation including the ``REWIND SOURCE``
statement woulnd't work on the source.

Due to its nature, a stream isn't often resumable. A resumable source is
mostly used for relatively static data sources such as relations or files.
Also, because implementing the ``RewindableSource`` interface is even harder
than implementing the ``Resumable`` interface, utilities are usually used.

.. _server_programming_go_sources_utilities:

Utilities
---------

There're some utilities to support implementing sources and its creators. This
subsection describes each utility.

``core.ImplementSourceStop``
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

``core.ImplementSourceStop`` is a function that implements the ``Stop`` method
of a source in a thread-safe manner::

    func ImplementSourceStop(s Source) Source

A source returned from this function is resumable, but not rewindable even if
the original source implements the ``core.RewindableSource`` interface. In
addition, although a source passed to ``core.ImplementSourceStop`` can
explicitly implement the ``core.Resumable`` interface, its ``Pause`` and
``Resume`` method will never be called because the source returned from
``core.ImplementSourceStop`` also implements those methods and controls
pause and resume.

To apply this function, a source must satisfy following restrictions:

1. The ``GenerateStream`` method must be implemented in a way that it can safely
   be called again after it returns.
2. The ``GenerateStream`` method must return when the ``core.Writer.Write``
   returned ``core.ErrSourceStopped``. The method must return exactly the same
   error variable that the writer returned.
3. The ``Stop`` method just returns nil.

    * This means all resource allocation and deallocation must be done within
      the ``GenerateStream`` method.

A typical implementation of a source passed to ``core.ImplementSourceStop`` is
shown below::

    func (s *MySource) GenerateStream(ctx *core.Context, w core.Writer) error {
        <initialization>
        defer func() {
            <clean up>
        }()

        for {
            t := <create a new tuple>
            if err := w.Write(ctx, t); err != nil {
                return err
            }
        }
        return nil
    }

    func (s *MySource) Stop(ctx *core.Context) error {
        return nil
    }

If a source wants to ignore errors returned from ``core.Writer.Write`` other
than ``core.ErrSourceStopped``, the ``GenerateStream`` method can be modified
as::

    if err := w.Write(ctx, t); err != nil {
        if err == core.ErrSourceStopped {
            return err
        }
    }

By applying ``core.ImplementSourceStop``, the ``Ticker`` above can be
implemented as follows::

    type Ticker struct {
        interval time.Duration
    }

    func (t *Ticker) GenerateStream(ctx *core.Context, w core.Writer) error {
        var cnt int64
        for ; ; cnt++ {
            tuple := core.NewTuple(data.Map{"tick": data.Int(cnt)})
            if err := w.Write(ctx, tuple); err != nil {
                return err
            }
            time.Sleep(t.interval)
        }
        return nil
    }

    func (t *Ticker) Stop(ctx *core.Context) error {
        return nil
    }

    type TickerCreator struct {
    }

    func (t *TickerCreator) CreateSource(ctx *core.Context,
        ioParams *bql.IOParams, params data.Map) (core.Source, error) {
        interval := 1 * time.Second
        if v, ok := params["interval"]; ok {
            i, err := data.ToDuration(v)
            if err != nil {
                return nil, err
            }
            interval = i
        }
        return core.ImplementSourceStop(&Ticker{
            interval: interval,
        }), nil
    }

There's no ``stopped`` flag now. In this version, the ``Stop`` method of the
source returned by ``core.ImplementSourceStop`` waits until the
``GenerateStream`` method returns.

``core.NewRewindableSource``
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

``core.NewRewindableSource`` is a function that converts a regular source into
a rewindable source::

    func NewRewindableSource(s Source) RewindableSource

A source returned from this function is resumable and rewindable. A source
passed to the function needs to satisfy the same restrictions as
``core.ImplementSourceStop``. In addition to that, there's one more restriction
for ``core.NewRewindableSource``:

4. The ``GenerateStream`` method must return when the ``core.Writer.Write``
   returned ``core.ErrSourceRewound``. The method must return exactly the same
   error variable that the writer returned.

Although the ``GenerateStream`` method of a rewindable source must not return
after it emits all tuples, a source passed to the ``core.NewRewindableSource``
function needs to return in that situation. For example, let's assume there's a
source that generate tuples from each line in a file. To implement the source
without a help of the utility function, its ``GenerateStream`` must wait for
the ``Rewind`` method to be called after it processes all lines in the file.
However, with the utility, its ``GenerateStream`` can just return once it emits
all tuples. Therefore, a typical implementation of a source passed to the
utility can be same as a source for ``core.ImplementSourceStop``.

As it will be shown later, a source that infinitely emits tuples can also be
rewindable in some sense.

The following is an example of ``TickerCreator`` modified from the example for
``core.ImplementSourceStop``::

    func (t *TickerCreator) CreateSource(ctx *core.Context,
        ioParams *bql.IOParams, params data.Map) (core.Source, error) {
        interval := 1 * time.Second
        if v, ok := params["interval"]; ok {
            i, err := data.ToDuration(v)
            if err != nil {
                return nil, err
            }
            interval = i
        }

        rewindable := false
        if v, ok := params["rewindable"]; ok {
            b, err := data.AsBool(v)
            if err != nil {
                return nil, err
            }
            rewindable = b
        }

        src := &Ticker{
            interval: interval,
        }
        if rewindable {
            return core.NewRewindableSource(src), nil
        }
        return core.ImplementSourceStop(src), nil
    }

In this example, ``Ticker`` has the ``rewindable`` parameter. If it is true,
the source becomes rewindable::

    CREATE SOURCE my_rewindable_ticker TYPE ticker WITH rewindable = true;

By issuing the ``REWIND SOURCE`` statement, ``my_rewindable_ticker`` resets
the value of ``tick`` field::

    REWIND SOURCE my_rewindable_ticker;

    -- output examples of SELECT RSTREAM * FROM my_rewindable_ticker [RANGE 1 TUPLES];
    {"tick":0}
    {"tick":1}
    {"tick":2}
    ...
    {"tick":123}
    -- REWIND SOURCE is executed here
    {"tick":0}
    {"tick":1}
    ...

``bql.SourceCreatorFunc``
^^^^^^^^^^^^^^^^^^^^^^^^^

``bql.SourceCreatorFunc`` is a function that converts a function having the
same signature as ``SourceCreator.CreateSource`` to a ``SourceCreator``::

    func SourceCreatorFunc(f func(*core.Context,
        *IOParams, data.Map) (core.Source, error)) SourceCreator

For example, ``TickerCreator`` above and its registration can be modified to as
follows with this utility::

    func CreateTicker(ctx *core.Context,
        ioParams *bql.IOParams, params data.Map) (core.Source, error) {
        interval := 1 * time.Second
        if v, ok := params["interval"]; ok {
            i, err := data.ToDuration(v)
            if err != nil {
                return nil, err
            }
            interval = i
        }
        return core.ImplementSourceStop(&Ticker{
            interval: interval,
        }), nil
    }

    func init() {
        bql.MustRegisterGlobalSourceCreator("ticker",
            bql.SourceCreatorFunc(CreateTicker))
    }

A Complete Example
------------------

A complete example of ``Ticker`` is shown in this subsection. Assume that the
import path of the example is ``github.com/sensorbee/examples/ticker``, which
doesn't actually exist. There're two files in the repository:

* ticker.go
* plugin/plugin.go

The example uses ``core.NewRewindableSource`` utility function.

ticker.go
^^^^^^^^^

::

    package ticker

    import (
        "time"

        "gopkg.in/sensorbee/sensorbee.v0/bql"
        "gopkg.in/sensorbee/sensorbee.v0/core"
        "gopkg.in/sensorbee/sensorbee.v0/data"
    )

    type Ticker struct {
        interval time.Duration
    }

    func (t *Ticker) GenerateStream(ctx *core.Context, w core.Writer) error {
        var cnt int64
        for ; ; cnt++ {
            tuple := core.NewTuple(data.Map{"tick": data.Int(cnt)})
            if err := w.Write(ctx, tuple); err != nil {
                return err
            }
            time.Sleep(t.interval)
        }
        return nil
    }

    func (t *Ticker) Stop(ctx *core.Context) error {
        // This method will be implemented by utility functions.
        return nil
    }

    type TickerCreator struct {
    }

    func (t *TickerCreator) CreateSource(ctx *core.Context,
        ioParams *bql.IOParams, params data.Map) (core.Source, error) {
        interval := 1 * time.Second
        if v, ok := params["interval"]; ok {
            i, err := data.ToDuration(v)
            if err != nil {
                return nil, err
            }
            interval = i
        }

        rewindable := false
        if v, ok := params["rewindable"]; ok {
            b, err := data.AsBool(v)
            if err != nil {
                return nil, err
            }
            rewindable = b
        }

        src := &Ticker{
            interval: interval,
        }
        if rewindable {
            return core.NewRewindableSource(src), nil
        }
        return core.ImplementSourceStop(src), nil
    }

plugin/plugin.go
^^^^^^^^^^^^^^^^

::

    package plugin

    import (
        "gopkg.in/sensorbee/sensorbee.v0/bql"

        "github.com/sensorbee/examples/ticker"
    )

    func init() {
        bql.MustRegisterGlobalSourceCreator("ticker", &ticker.TickerCreator{})
    }
