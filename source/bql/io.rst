*****************************
Input/Output/State Definition
*****************************

To process streams of data, they need to be imported into SensorBee and
the processing results have to be exported from it. This chapter introduces
input and output components in BQL.

This chapter also describes how BQL supports stateful data processing using
user-defined staets (UDSs).

TODO:

- Data Output
    - CREATE SINK
    - UPDATE SINK
    - DROP SINK
- Stateful Data Processing
    - CREATE STATE
    - UPDATE STATE
    - DROP STATE
    - LOAD STATE
    - SAVE STATE

.. _bql_io_data_input:

Data Input
==========

BQL inputs a stream of data from a **source**. A source connects to a stream
defined and generated outside SensorBee, input data from the stream, converts
the data into tuples, and finally emits tuples for further processing. This
section describes how a source can be created, operated, and dropped.

Creating a Source
-----------------

A source can be created by the :ref:`ref_stmts_create_source` statement.

::

    CREATE SOURCE logs TYPE file WITH path = "access_log.jsonl";

In this example, a source named ``logs`` is created and it has the type
``file``. The ``file`` type has a required parameter called ``path``. The
parameter is specified in the ``WITH`` clause. Once a source is created, other
nodes described later can input tuples from the source and compute results
based on them.

When multiple parameters are required, they should be separated by commas::

    CREATE SOURCE src TYPE some_type
        WITH param1 = val1, param2 = val2, param3 = val3;

Source types can be registered to the SensorBee server as plugins. To learn
how to develop and register a source plugin, see
:ref:`server_programming_go_sources`.

Built-in Sources
^^^^^^^^^^^^^^^^

BQL has built-in source types.

``file``

    The ``file`` type provides a source that input tuples from an existing file.

``node_statuses``

    The ``node_statuses`` source periodically emits tuples having information of
    nodes in a topology. The status includes connected nodes, number of tuples
    emitted from or written to the node, and so on.

``edge_statuses``

    The ``edge_statuses`` source periodically emits tuples having information of
    each edge (a.k.a. pipe) that connects a pair of nodes. Although this
    information is contained in tuples emitted from ``node_statuses`` source,
    the ``edge_statuses`` source provides more edge-centric view of IO statuses.

``dropped_tuples``

    The ``dropped_tuples`` emits tuples dropped from a topology. It only reports
    once per tuple. Tuples are often dropped from a topology because a source or
    a stream is not connected to any other node or a ``SELECT`` statement tries
    to look up a nonexistent field of a tuple.

Further information of each source can be found in the reference (TODO: link).

Pausing and Resuming a Source
-----------------------------

By default, a source starts emitting tuples as soon as it is created. By adding
``PAUSED`` keyword to the ``CREATE SOURCE`` statement, it creates a source
paused at its startup::

    CREATE PAUSED SOURCE logs TYPE file WITH path = "access_log.jsonl";

The :ref:`ref_stmts_resume_source` statement makes a paused source emit tuples
again::

    RESUME SOURCE logs;

The statement takes the name of the source to be resumed.

A source can be paused after it is created by the :ref:`ref_stmts_pause_source`
statement::

    PAUSE SOURCE logs;

The statement also takes the name of the source to be paused.

Not all sources support ``PAUSE SOURCE`` and ``RESUME SOURCE`` statements.
Issuing statements to those sources results in an error.

Rewinding a Source
------------------

Some sources can be rewound, that is, they emit tuples that they have emitted
again. The :ref:`ref_stmts_rewind_source` rewinds a source if the source
supports the statement::

    REWIND SOURCE logs;

The statement takes the name of the source to be rewound. Issuing the statement
to sources that don't support rewinding results in an error.

Dropping a Source
-----------------

The :ref:`ref_stmts_drop_source` statement drops (i.e. removes) a source from
a topology::

    DROP SOURCE logs;

The statement takes the name of the source to be dropped. Other nodes in a
topology cannot refer to the source once it's dropped. Also, nodes connected to
a source may cascadingly be stopped when the source gets dropped.

.. _bql_io_data_output:

Data Output
===========

.. _bql_io_state:

Stateful Data Processing
========================
