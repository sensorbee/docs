*****************************
Input/Output/State Definition
*****************************

To process streams of data, they need to be imported into SensorBee and
the processing results have to be exported from it. This chapter introduces
input and output components in BQL.

This chapter also describes how BQL supports stateful data processing using
user-defined staets (UDSs).

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

Each source type has its own parameters and there's no parameter that is common
to all sources.

Source types can be registered to the SensorBee server as plugins. To learn
how to develop and register a source plugin, see
:ref:`server_programming_go_sources`.

Built-in Sources
^^^^^^^^^^^^^^^^

BQL has built-in source types.

``file``

    The ``file`` type provides a source that inputs tuples from an existing file.

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

Results of processing tuples need to be emitted systems or services running
outside the SensorBee server so that it can work with them as a part of a
large system. A **sink** outputs the result of computations performed within
the SensorBee server. This section explains how sinks are operated in BQL.

Creating a Sink
---------------

A sink can be created by the :ref:`ref_stmts_create_sink` statement::

    CREATE SINK filtered_logs TYPE file WITH path = "filtered_access_log.jsonl";

The statement is very similar to the ``CREATE SOURCE`` statement. It takes the
name of the new sink, its type, and parameters. Multiple parameters can also be
provided as a list separated by commas. Each sink type has its own parameters
and there's no parameter that is common to all sinks.

Sink types can also be registered to the SensorBee server as plugins. To learn
how to develop and register a sink plugin, see
:ref:`server_programming_go_sink`.

Built-in Sinks
^^^^^^^^^^^^^^

BQL has built-in sink types.

``file``

    The ``file`` type provides a sink that writes tuples to a file.

``stdout``

    The ``stdout`` sinks writes output tuples to stdout.

``uds``

    The ``uds`` sink passes tuples to user-defined states, which is described
    later.

Writing Data to a Sink
-----------------------

The :ref:`ref_stmts_insert_into` statement writes data to a sink::

    INSERT INTO filtered_logs FROM filtering_stream;

The statement takes the name of sink to be written and the name of a source or
a stream, which will be described in following chapters.

Dropping a Sink
---------------

The :ref:`ref_stmts_drop_sink` statement drops a sink from a topology::

    DROP SINK filtered_logs;

The statement taks the name of the sink to be dropped. The sink cannot be
accessed once it gets dropped. All ``INSERT INTO`` statements writing to the
dropped sink are also stopped.

.. _bql_io_state:

Stateful Data Processing
========================

SensorBee supports user-defined states (UDSs) to perform stateful streaming
data processing. Such processing includes not only aggregates such as counting
but also machine learning, adaptive sampling, and so on. In natural language
processing, dictionaries or configurations for tokenizers can also be considered
as states.

This section describes operations among UDSs. Use cases of UDSs described in
:ref:`tutorial` tutorials and how to develop a custom UDS is explained in the
:ref:`server programming <server_programming_go_states>` part.

Creating a UDS
--------------

A UDS can be created by the :ref:`bql_stmts_create_state` statement::

    CREATE STATE age_classifier TYPE jubaclassifier_arow
        WITH label_field = "age", regularization_weight = 0.001;

This statement creates a UDS named ``age_classifier`` with the type
``jubaclassifier_arow``. It has two parameters: ``label_field`` and
``regularization_weight``. Each UDS type has its own parameters and there's no
parameter that is common to all UDS.

A UDS is usually used by user-defined functions (UDFs) that knows about its
specific UDS type. See :ref:`server programming <server_programming_go_states>`
part for details.

Saving a State
--------------

The :ref:`ref_stmts_save_state` saves a UDS::

    SAVE STATE age_classifier;

The statement takes the name of the UDS to be saved. After the statement is
issued, SensorBee saves the state based on the given configuration. The location
and the format of saved data depend on the configuration and are unknown to
users.

The ``SAVE STATE`` statement may take a ``TAG`` to support versioning of the
saved data::

    SAVE STATE age_classifier TAG initial;
    -- or
    SAVE STATE age_classifier TAG trained;

When the ``TAG`` clause is omitted, ``default`` will be the default tag name.

Loading a State
---------------

The :ref:`ref_stmts_load_state` loads a UDS that was previously saved by the
``SAVE STATE`` statement::

    LOAD STATE age_classifier TYPE jubaclassifier_arow;

The statement takes the name of the UDS to be loaded and its type name. it fails
if the UDS hasn't been saved yet.

The ``LOAD STATE`` statements may also take a ``TAG``::

    LOAD STATE age_classifier TYPE jubaclassifier_arow TAG initial;
    -- or
    LOAD STATE age_classifier TYPE jubaclassifier_arow TAG trained;

The UDS needs to have been saved with the specified tag before. When the ``TAG``
clause is omitted, it's same as::

    LOAD STATE age_classifier TYPE jubaclassifier_arow TAG default;

The ``LOAD STATE`` statement supports the ``OR CREATE IF NOT SAVED`` clause.
When the clause is given, the statement tries to create a UDS if it hasn't been
saved yet::

    LOAD STATE age_classifier TYPE jubaclassifier_arow
        OR CREATE IF NOT SAVED
            WITH label_field = "age", regularization_weight = 0.001;

It just load the UDS and doesn't create a new UDS if there's the previously
saved data. The ``OR CREATE IF NOT SAVED`` clause can be used with the ``TAG``
clause::

    LOAD STATE age_classifier TYPE jubaclassifier_arow TAG trained
        OR CREATE IF NOT SAVED
            WITH label_field = "age", regularization_weight = 0.001;

Dropping a State
----------------

The :ref:`ref_stmts_drop_state` statement drop a UDS from a topology::

    DROP STATE age_classifier;

The statement takes the name of the UDS to be dropped. Once a UDS is dropped, it
will no longer be referred by any statement unless the UDS is cached somewhere.
