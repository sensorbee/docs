.. _server_programming_go_states:

User-Defined States
===================

This section describes how to write a UDS in Go.

Implementing a UDS
------------------

A struct implementing the following interface can be used as a UDS:

.. code-block:: go

    type SharedState interface {
        Terminate(ctx *Context) error
    }

.. todo:: Add a link to godoc.org

This interface is defined in ``gopkg.in/sensorbee/sensorbee.v0/core`` package.

``Terminate`` method is called when the UDS becomes no longer in use. It should
release any resource that the UDS has allocated so far.

As an example, a UDS having a monotonically increasing counter can be
implemented as follows:

.. code-block:: go

    type Counter struct {
        c int64
    }

    func (c *Counter) Terminate(ctx *core.Context) error {
        return nil
    }

    func (c *Counter) Next() int64 {
        return atomic.AddInt64(&c.c, 1)
    }

At the moment, there's no way to manipulate the UDS from BQL statements. UDSs
are usually provided with a set of UDFs that read or update the UDS. It'll be
described later in :ref:`server_programming_go_states_manipulation`. Before
looking into the UDS manipulation, registering and creating a UDS needs to be
explained.

Registering a UDS
-----------------

To register a UDS to the SensorBee server , the UDS needs to provide its
``UDSCreator``. ``UDSCreator`` is an interface defined in
``gopkg.in/sensorbee/sensorbee.v0/bql/udf`` package as follows:

.. code-block:: go

    type UDSCreator interface {
        CreateState(ctx *core.Context, params data.Map) (core.SharedState, error)
    }

``UDSCreator.CreateState`` method is called when executing a ``CREATE STATE``
statement. The method creates a new instance of the UDS and initializes it with
the given parameters. The argument ``ctx`` has the processing context
information and ``params`` has parameters specified in the ``WITH`` clause of
the ``CREATE STATE``.

The creator can be registered by ``RegisterGlobalUDSCreator`` or
``MustRegisterGlobalUDSCreator`` function defined in
``gopkg.in/sensorbee/sensorbee.v0/bql/udf`` package.

The following is the implementation and the registration of the creator for
``Counter`` UDS above:

.. code-block:: go

    type CounterCreator struct {
    }

    func (c *CounterCreator) CreateState(ctx *core.Context,
        params data.Map) (core.SharedState, error) {
        cnt := &Counter{}
        if v, ok := params["start"]; ok {
            i, err := data.ToInt(v)
            if err != nil {
                return nil, err
            }
            cnt.c = i - 1
        }
        return cnt, nil
    }

    func init() {
        udf.MustRegisterGlobalUDSCreator("my_counter", &CounterCreator{})
    }

The creator in this example is registered with the UDS type name ``my_counter``.
The creator supports ``start`` parameter which is used as the first value that
``Counter.Next`` returns. The parameter can be specified in the ``CREATE STATE``
as follows::

    CREATE STATE my_counter_instance TYPE my_counter WITH start = 100;

Because the creator creates a new instance every time the ``CREATE STATE`` is
executed, there can be multiple instances of a specific UDS type::

    CREATE STATE my_counter_instance1 TYPE my_counter;
    CREATE STATE my_counter_instance2 TYPE my_counter;
    CREATE STATE my_counter_instance3 TYPE my_counter;
    ...

Once an instance of the UDS is created by the ``CREATE STATE``, UDFs can refer
them and manipulate their state.

``udf.UDSCreatorFunc``
^^^^^^^^^^^^^^^^^^^^^^

A function having the same signature as ``UDSCreator.CreateState`` can be
converted into ``UDSCreator`` by by ``udf.UDSCreatorFunc`` utility function:

.. code-block:: go

    func UDSCreatorFunc(f func(*core.Context, data.Map) (core.SharedState, error)) UDSCreator

For example, ``CounterCreator`` can be defined as a function and registered as
follows with this utility:

.. code-block:: go

    func CreateCounter(ctx *core.Context,
        params data.Map) (core.SharedState, error) {
        cnt := &Counter{}
        if v, ok := params["start"]; ok {
            i, err := data.ToInt(v)
            if err != nil {
                return nil, err
            }
            cnt.c = i - 1
        }
        return cnt, nil
    }

    func init() {
        udf.MustRegisterGlobalUDSCreator("my_counter",
            &udf.UDSCreatorFunc(CreateCounter))
    }

To support ``SAVE STATE`` and ``LOAD STATE`` statements, however, this utility
function cannot be used because the creator needs to have the ``LoadState``
method. How to support saving and loading is described later.

.. _server_programming_go_states_manipulation:

Manipulating a UDS via a UDF
----------------------------

To manipulate a UDS from BQL statements, a set of UDFs that read or update the
UDS has to be provided with it:

.. code-block:: go

    func Next(ctx *core.Context, uds string) (int64, error) {
        s, err := ctx.SharedStates.Get(uds)
        if err != nil {
            return 0, err
        }

        c, ok := s.(*Counter)
        if !ok {
            return 0, fmt.Errorf("the state isn't a counter: %v", uds)
        }
        return c.Next(), nil
    }

    func init() {
        udf.MustRegisterGlobalUDF("my_next_count", udf.MustConvertGeneric(Next))
    }

In this example, a UDF ``my_next_count`` is registered to the SensorBee server.
The UDF calls ``Counter.Next`` method to obtain the next count and returns it.
The UDF receives one argument ``uds`` that is the name of the UDS to be updated.

::

    CREATE STATE my_counter_instance TYPE my_counter;
    CREATE STREAM events_with_id AS
        SELECT RSTREAM my_next_count("my_counter_instance") AS id, *
        FROM events [RANGE 1 TUPLES];

The BQL statements above add IDs to tuples emitted from a stream ``events``. The
state ``my_counter_instance`` is created with the type ``my_counter``. Then,
``my_next_count`` UDF is called with the name. Every time the UDF is called,
the state of ``my_counter_instance`` is updated by its ``Next`` method.

``my_next_count`` (i.e. ``Next`` function in Go) can look up the instance of
the UDS by its name through ``core.Context.SharedStates``. ``SharedStates``
manages all the UDSs created in a topology. ``SharedState.Get`` returns the
instance of the UDS having the given name. It returns an error if it couldn't
find the instance. In the example above, ``my_next_count("my_counter_instance")``
will look up an instance of the UDS having the name ``my_counter_instance``,
which was previously created by the ``CREATE STATE``. The UDS returned from
``Get`` method has the type ``core.SharedState`` and cannot directly be used as
``Counter``. Therefore, it has to be cast to ``*Counter``.

Since the state can be any type satisfying ``core.SharedState``, a UDS can
potentially have any information such as machine learning models,
dictionaries for natural language processing, or even an in-memory database.

.. note::

    As UDFs are concurrently called from multiple goroutines, UDSs also needs
    to be thread-safe.

Saving and Loading a UDS
------------------------

``Counter`` implemented so far doesn't support saving and loading its state.
Thus, its count will be reset every time the server restarts. To save the
state and load it later on, the UDS and its creator need to provide some
methods. After providing those method, the state can be saved by the
``SAVE STATE`` statement and loaded by ``LOAD STATE`` statement.

Supporting ``SAVE STATE``
^^^^^^^^^^^^^^^^^^^^^^^^^

By adding ``Save`` method having the following signature to a UDS, the UDS can
be saved by the ``SAVE STATE`` statement:

.. code-block:: go

    Save(ctx *core.Context, w io.Writer, params data.Map) error

``Save`` method writes all the data that the state has to ``w io.Writer``.
The data can be written in any format as long as corresponding loading methods
can reconstruct the state from it. It can be in JSON, msgpack, Protocol Buffer,
and so on.

.. warning::

    Providing forward/backward compatibility or version controlling of the saved
    data is the responsibility of the author of the UDS.

``*core.Context`` has the processing context information. ``params`` argument
is not used at the moment and reserved for the future use.

Once Save method is provided, the UDS can be saved by ``SAVE STATE`` statement::

    SAVE STATE my_counter_instance;

The ``SAVE STATE`` doesn't take any parameters now. The location and the
physical format of the saved UDS data depend on the configuration of the
SensorBee server or program running BQL statements. However, it is guaranteed
that the saved data can be loaded by the same program via the ``LOAD STATE``
statement, which is described later.

``Save`` method of previously implemented ``Counter`` can be as follows:

.. code-block:: go

    func (c *Counter) Save(ctx *core.Context, w io.Writer, params data.Map) error {
        return binary.Write(w, binary.LittleEndian, atomic.LoadInt64(&c.c))
    }

.. note::

    Because this counter is very simple, there's no version controlling logic
    in the method. As the minimum solution, having a version number at the
    beginning of the data is sufficient for most cases.

Supporting ``LOAD STATE``
^^^^^^^^^^^^^^^^^^^^^^^^^

To support the ``LOAD STATE`` statement, a ``UDSCreator`` needs to have
``LoadState`` method having the following signature:

.. code-block:: go

    LoadState(ctx *core.Context, r io.Reader, params data.Map) (core.SharedState, error)

.. note::

    ``LoadState`` method needs to be defined in a ``UDSCreator``, not in the
    UDS itself.

``LoadState`` method reads data from ``r io.Reader``. The data has exactly the
same format as the one previously written by ``Save`` method of a UDS.
``params`` has parameters specified in the ``SET`` clause in the ``LOAD STATE``
statement.

.. note::

    Parameters specified in the ``SET`` clause doesn't have to be same as ones
    given in the ``WITH`` clause of the ``CREATE STATE`` statement. See
    :ref:`ref_stmts_load_state` for details.

When ``LoadState`` method returns an error, the ``LOAD STATE`` statement with
``CREATE IF NOT STATE`` doesn't fallback to ``CREATE STATE``, but it just fails.

Once ``LoadState`` method is added to the ``UDSCreator``, the saved state can be
loaded by ``LOAD STATE`` statement.

``LoadState`` method of previously implemented ``CounterCreator`` can be as
follows:

.. code-block:: go

    func (c *CounterCreator) LoadState(ctx *core.Context, r io.Reader,
        params data.Map) (core.SharedState, error) {
        cnt := &Counter{}
        if err := binary.Read(r, binary.LittleEndian, &cnt.c); err != nil {
            return nil, err
        }
        return cnt, nil
    }

Providing ``Load`` method in a UDS
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

In addition to implementing ``LoadState`` method in a UDS's creator, a UDS
itself can provide ``Load`` method. While ``LoadState`` method creates a new
state instance and replace it with the previous instance, ``Load`` method
dynamically modifies the existing instance. Therefore, ``Load`` method can
potentially be more efficient than ``LoadState`` method although it has to
provide appropriate failure handling and concurrency control so that (1) the
UDS doesn't become invalid on failure (i.e. ``Load`` methods is "exception
safe") or by concurrent calls, and (2) other operations on the UDS don't block
for a long time.

The signature of ``Load`` method is almost the same as ``LoadState`` method
except that ``Load`` method doesn't return a new ``core.SharedState`` but
updates the UDS itself instead:

.. code-block:: go

    Load(ctx *Context, r io.Reader, params data.Map) error

``Load`` method of previously implemented ``Counter`` can be as follows:

.. code-block:: go

    func (c *Counter) Load(ctx *core.Context, r io.Reader, params data.Map) error {
        var cnt int64
        if err := binary.Read(r, binary.LittleEndian, &cnt); err != nil {
            return err
        }
        atomic.StoreInt64(&c.c, cnt)
        return nil
    }

How Loading is Processed
^^^^^^^^^^^^^^^^^^^^^^^^

SensorBee tries to use these two loading methods ``LoadState`` and ``Load``
in the following rule:

#. When a UDS's creator doesn't provide ``LoadState`` method, the ``LOAD STATE``
   statement fails.

    * The ``LOAD STATE`` statement fails even if the UDS implements its ``Load``
      method. To support the statement, ``LoadState`` method is always required
      in its creator. This is because ``Load`` method only works when an
      instance of the UDS is already created or loaded, and it cannot be used
      for a nonexistent instance.
    * The ``LOAD STATE CREATE IF NOT SAVED`` statement also fails if
      ``LoadState`` method isn't provided. The statement calls ``CreateState``
      method when the state hasn't previously been saved. Otherwise, it'll try
      to load the saved data. Therefore, if the data is previously saved and
      an instance of the UDS hasn't been created yet, the statement cannot
      create a new instance without ``LoadState`` method in the creator. To be
      consistent on various conditions, the ``LOAD STATE CREATE IF NOT SAVED``
      statement fails if ``LoadState`` method isn't provided regardless of
      whether the state has been saved before or not.

#. When a UDS's creator provides ``LoadState`` method and the UDS doesn't
   provide ``Load`` method, the ``LOAD STATE`` statement tries to load a model
   through ``LoadState`` method.

    * It will create a new instance so that it consumes twice as much memory.

#. When a UDS's creator provides ``LoadState`` method and the UDS also provides
   ``Load`` method,

    * ``Load`` method will be used when the instance has already been created or
      loaded.

        * ``LoadState`` method wouldn't be used even if ``Load`` method failed.

    * ``LoadState`` method will be used otherwise.

.. note::

    This is already mentioned in the list above, but ``LoadState`` method always
    needs to be provided even if a UDS implements ``Load`` method.

A Complete Example
------------------

A complete example of the state is shown in this subsection. Assume that the
import path of the example repository is
``github.com/sensorbee/examples/counter``, which doesn't actually exist. The
repository has two files:

* counter.go
* plugin/plugin.go

counter.go
^^^^^^^^^^

.. code-block:: go

    package counter

    import (
        "encoding/binary"
        "fmt"
        "io"
        "sync/atomic"

        "gopkg.in/sensorbee/sensorbee.v0/core"
        "gopkg.in/sensorbee/sensorbee.v0/data"
    )

    type Counter struct {
        c int64
    }

    func (c *Counter) Terminate(ctx *core.Context) error {
        return nil
    }

    func (c *Counter) Next() int64 {
        return atomic.AddInt64(&c.c, 1)
    }

    func (c *Counter) Save(ctx *core.Context, w io.Writer, params data.Map) error {
        return binary.Write(w, binary.LittleEndian, atomic.LoadInt64(&c.c))
    }

    func (c *Counter) Load(ctx *core.Context, r io.Reader, params data.Map) error {
        var cnt int64
        if err := binary.Read(r, binary.LittleEndian, &cnt); err != nil {
            return err
        }
        atomic.StoreInt64(&c.c, cnt)
        return nil
    }

    type CounterCreator struct {
    }

    func (c *CounterCreator) CreateState(ctx *core.Context,
        params data.Map) (core.SharedState, error) {
        cnt := &Counter{}
        if v, ok := params["start"]; ok {
            i, err := data.ToInt(v)
            if err != nil {
                return nil, err
            }
            cnt.c = i - 1
        }
        return cnt, nil
    }

    func (c *CounterCreator) LoadState(ctx *core.Context, r io.Reader,
        params data.Map) (core.SharedState, error) {
        cnt := &Counter{}
        if err := binary.Read(r, binary.LittleEndian, &cnt.c); err != nil {
            return nil, err
        }
        return cnt, nil
    }

    func Next(ctx *core.Context, uds string) (int64, error) {
        s, err := ctx.SharedStates.Get(uds)
        if err != nil {
            return 0, err
        }

        c, ok := s.(*Counter)
        if !ok {
            return 0, fmt.Errorf("the state isn't a counter: %v", uds)
        }
        return c.Next(), nil
    }

plugin/plugin.go
^^^^^^^^^^^^^^^^

.. code-block:: go

    package plugin

    import (
        "gopkg.in/sensorbee/sensorbee.v0/bql/udf"

        "github.com/sensorbee/examples/counter"
    )

    func init() {
        udf.MustRegisterGlobalUDSCreator("my_counter",
            &counter.CounterCreator{})
        udf.MustRegisterGlobalUDF("my_next_count",
            udf.MustConvertGeneric(counter.Next))
    }


Writing Tuples to a UDS
-----------------------

When a UDS implements ``core.Writer``, the ``INSERT INTO`` statement can
insert tuples into the UDS via the ``uds`` sink:

.. code-block:: go

    type Writer interface {
        Write(*Context, *Tuple) error
    }

The following is the example of using the ``uds`` sink::

    CREATE STATE my_state TYPE my_state_type;
    CREATE SINK my_state_sink TYPE uds WITH name = "my_state";
    INSERT INTO my_state_sink FROM some_stream;

If ``my_state_type`` doesn't implement ``core.Writer``, the ``CREATE SINK``
statement fails. Every time ``some_stream`` emits a tuple, the ``Write``
method of ``my_state`` is called.

Example
^^^^^^^

Models provided by Jubatus machine learning plugin for SensorBee implement
the ``Write`` method. When tuples are inserted into a UDS, it trains the model
it has.
