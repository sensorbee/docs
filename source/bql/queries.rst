*******
Queries
*******

The previous chapters described how to define data sources and sinks to communicate with the outside world.
Now we discuss how to transform the data stream from those sources and write it to the defined sinks.

Processing Model
================

Overview
--------

The processing model in BQL is similar to what is explained in [cql]_.
In this model, each tuple in a stream has the shape :math:`(t, d)`, where :math:`t` is the original timestamp and :math:`d` the data contained.

In order to execute SQL-like queries, we need to obtain a finite set of tuples from the possibly unbounded stream, a *relation*.
In the processing step at time :math:`t^*`, we use a *stream-to-relation* operator :math:`R` that converts a certain set of tuples in the stream to a relation :math:`R(t^*)`.
This relation is then processed with a *relation-to-relation* operator :math:`O` that is expressed in a form very closely related to SQL.
Finally, a *relation-to-stream* operator :math:`S` will emit certain rows from the output relation :math:`O(R(t^*))` into the output stream, possibly taking into account the results of the previous execution step :math:`O(R(t^*_{\text{prev}}))`.

.. [cql] Arasu et al., "The CQL Continuous Query Language: Semantic Foundations and Query Execution", http://ilpubs.stanford.edu:8090/758/1/2003-67.pdf

.. [streamsql] Jain et al., "Towards a Streaming SQL Standard", http://cs.brown.edu/~ugur/streamsql.pdf

This three-step pipeline is executed for each tuple, but only for one tuple at a time.
Therefore, during execution there is a well-defined "current tuple".
This also means that if there is no tuple in the input stream for a long time, transformation functions will not be called.

We will now explain the kind of stream-to-relation and relation-to-stream operators that can be used.


Stream-to-Relation Operators
----------------------------

In BQL, there are two different stream-to-relation operators, a time-based one and a tuple-based one.
We also call them "window operators", since they define a sliding window on the input stream.
In terms of BQL, the window operator is given after a stream name in the ``FROM`` clause within brackets and using the ``RANGE`` keyword, for example::

    ... FROM events [RANGE 5 SECONDS] ...
    ... FROM data [RANGE 10 TUPLES] ...
    ... FROM left [RANGE 2 SECONDS], right [RANGE 5 TUPLES] ...

From an SQL point of view, it makes sense to think of ``stream [RANGE window-spec]`` as the table to operate on.


The **time-based operator** is used with a certain time span :math:`I` (such as 60 seconds) and at point in time :math:`t^*` uses all tuples in the range :math:`[t^*-I, t^*]` to create the relation :math:`R(t^*)`.

Valid time spans are positive integer or float values, followed by the ``SECONDS`` or ``MILLISECONDS`` keyword, for example ``[RANGE 3.5 SECONDS]`` or ``[RANGE 200 MILLISECONDS]`` are valid specifications.

Notes:

- The point in time :math:`t^*` is *not* the "current time" (however that would be defined), but it is equal to the timestamp of the current tuple.
  This approach means that a stream can be reprocessed with identical results independent of the system clock of some server.
  Also it is not necessary to worry about a delay until a tuple arrives in the system and is processed there.
- It is assumed that the tuples in the input stream arrive in the order of their timestamps.
  If timestamps are out of order, the window contents are not well-defined.
- The sizes of relations :math:`R(t^*_1)` and :math:`R(t^*_2)` can be different, since there may be more or less tuples in the given time span.
  However, there is always at least one tuple in the relation (the current one).


The **tuple-based operator** is used with a number :math:`k` and uses the last :math:`k` tuples that have arrived (or *all* tuples that have arrived when this number is less than :math:`k`) to create the relation :math:`R(t^*)`.

Valid ranges are positive integral values, followed by the ``TUPLES`` keyword, for example ``[RANGE 10 TUPLES]`` is a valid specification.

Notes:

- The timestamps of tuples do not have any effect with this operator, they can also be out of order.
  Only the order in which the tuples arrived is important.
  (Note that for highly concurrent systems, "order" is not always a well-defined term.)
- At the beginning of stream processing, when less than :math:`k` tuples have arrived, the size of the relation will be less than :math:`k`.
  As soon as :math:`k` tuples have arrived, the relation size will be constant.


Relation-to-Stream operators
----------------------------

Once a resulting relation is computed, tuples in the relation needs to be
output as a stream again so that it can be referred by other ``SELECT``
statements as an input stream. To convert a relation to a stream, BQL provides
following three relation-to-stream operators:

* ``RSTREAM``
* ``ISTREAM``
* ``DSTREAM``

The following subsections describe how each operator works.

``RSTREAM`` operator
^^^^^^^^^^^^^^^^^^^^

When ``RSTREAM`` is specified, ``SELECT`` statements emits all tuples in
its relation. For example,

::

    SELECT RSTREAM * FROM src [RANGE 1 TUPLES];

this statement emits one tuple everytime it gets an input from ``src``.

::

    SELECT RSTREAM * FROM src [RANGE 100 TUPLES];

The statement above emits at most 100 tuples everytime a new tuples comes from ``src``.

::

    SELECT RSTREAM * FROM src1 [RANGE 10 TUPLES], src2 [RANGE 20 TUPLES];

This statement emits 200 tuples everytime it gets a new tuple from ``src1`` or
``src2``. This is because 10 tuples from ``src1`` and 20 tuples from ``src2``
are cross-joined and the resulting relation has 200 tuples in total.

``ISTREAM`` operator
^^^^^^^^^^^^^^^^^^^^

TODO: description


``DSTREAM`` operator
^^^^^^^^^^^^^^^^^^^^

TODO: description


The SELECT Clause
=================

TODO: description


Data Transformation
===================

TODO:

- CREATE STREAM
- DROP STREAM


Data Output
===========

TODO:

- INSERT INTO ... SELECT
- INSERT INTO ... FROM


Expression Evaluation
=====================

TODO:

- EVAL
