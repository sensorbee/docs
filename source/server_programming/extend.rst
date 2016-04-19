**************************************
Extending the SensorBee Server and BQL
**************************************

Many features of the server and BQL can be extended by plugins. This chapter
describes what types of features can be extended by users. Following chapters
in this part describes how to develop those features in specific programming
languages.

User-Defined Functions
======================

A user-defined function (UDF) is a function that is implemented by a user and
registered in the SensorBee server. Once it is registered, it be called from
BQL statements::

    SELECT RSTREAM my_udf(field) FROM src [RANGE 1 TUPLES];

A UDF behaves just like a built-in function. A UDF can take an arbitrary number
of arguments. Each argument can be any of :ref:`built-in types <bql_types_types>`
and can receive multiple types of values. A UDF can also support a variable-length
argument. A UDF has a single return value of any built-in type. When multiple return
values are required, a UDF can return the value as an ``array``.

.. note::

    BQL currently does not support ``CREATE FUNCTION`` statements like well-known
    RDBMSs. UDFs can only be added through Go programs.

.. todo:: describe volatility of functions

User-Defined Aggregate Functions
================================

A user-defined aggregate function (UDAF) is a function similar to a UDF but can
take aggregation parameters (see :ref:`bql_syntax_aggregates`) as arguments in
addition to regular arguments.

User-Defined Stream-Generating Functions
========================================

:ref:`Stream-generating functions <bql_stream_generating_functions>` can also
be user-defined. There are two types of UDSFs. The first type behaves like a source,
which is not connected to any input stream and generates tuples proactively::

    ... FROM my_counter() [RANGE ...

``my_counter`` above may emit tuples like ``{"count": 1}``.

This type of UDSFs are called source-like UDSFs.

The other type is called a stream-like UDSF and behaves just like a stream, which
receives tuples from one or more incoming streams or sources. It receives names
of streams or sources as its arguments::

    ... FROM my_udsf("another_stream", "yet_another_stream", other_params) [RANGE ...

Note that there is no rule on how to define UDFS's arguments. Thus, the order and
the use of arguments depend on each UDFS. For example, a UDFS might take an
``array`` of ``string`` containing names of input streams as its first argument:
``my_union(["stream1", "stream2", "stream3"])``. Names of input stream do not
even need to be located at the beginning of the argument list:
``my_udfs2(1, "another_stream")`` is also possible.

Using UDSFs is a very powerful way to extend BQL since they can potentially do
anything that the ``SELECT`` cannot do.

User-Defined States
===================

A user-defined state (UDS) can be provided to support stateful data processing
(see :ref:`bql_io_state`). A UDS is usually provided with a set of UDFs that
manipulate the state. Those UDFs take the name of the UDS as a ``string``
argument::

    CREATE STATE event_id_seq TYPE snowflake_id WITH machine_id = 1;
    CREATE STREAM events_with_id AS
        SELECT snowflake_id("event_id_seq"), * FROM events [RANGE 1 TUPLES];

In the example above, a UDS ``event_id_seq`` is created with the type
``snowflake_id``. Then, the UDS is passed to the UDF ``snowflake_id``, which
happens to have the same name as the type name of the UDS. The UDF looks up
the UDS ``event_id_seq`` and returns a value computed based on the state.

Source Plugins
==============

A source type developed by a user can be added to the SensorBee server as a plugin
so that it can be used in ``CREATE SOURCE`` statements. A source type can have any
number of required and optional parameters. Each parameter can have any of
:ref:`built-in types <bql_types_types>`.

Sink Plugins
============

A sink type developed by a user can be added to the SensorBee server as a plugin
so that it can be used in ``CREATE SINK`` statement. A sink type can have any
number of required and optional parameters. Each parameter can have any of
:ref:`built-in types <bql_types_types>`.
