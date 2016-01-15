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
This relation is then processed with a *relation-to-relation* operator :math:`O` that is expressed in a form very closely related to an SQL ``SELECT`` statement.
Finally, a *relation-to-stream* operator :math:`S` will emit certain rows from the output relation :math:`O(R(t^*))` into the output stream, possibly taking into account the results of the previous execution step :math:`O(R(t^*_{\text{prev}}))`.

This three-step pipeline is executed for each tuple, but only for one tuple at a time.
Therefore, during execution there is a well-defined "current tuple".
This also means that if there is no tuple in the input stream for a long time, transformation functions will not be called.

We will now explain the kind of stream-to-relation and relation-to-stream operators that can be used.


Stream-to-Relation Operators
----------------------------

In BQL, there are two different stream-to-relation operators, a time-based one and a tuple-based one.
We also call them "window operators", since they define a sliding window on the input stream.
In terms of BQL syntax, the window operator is given after a stream name in the ``FROM`` clause within brackets and using the ``RANGE`` keyword, for example::

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
- At the beginning of stream processing, when less than :math:`k` tuples have arrived, the size of the relation will be less than :math:`k`. [#fn_tuple-window]_
  As soon as :math:`k` tuples have arrived, the relation size will be constant.

.. [#fn_tuple-window] Sometimes this leads to unexpected effects or complicated workarounds, while the cases where this is a useful behavior may be few. Therefore this behavior may change in future version.


Relation-to-Stream Operators
----------------------------

Once a resulting relation :math:`O(R(t^*))` is computed, tuples in the relation need to be output as a stream again.
In BQL, there are three different relation-to-stream operators, ``RSTREAM``, ``ISTREAM`` and ``DSTREAM``.
We also call them "emit operators", since they control how tuples are emitted as output.
In terms of BQL syntax, the emit operator keyword is given after the ``SELECT`` keyword, for example::

    SELECT ISTREAM uid, msg FROM ...

The following subsections describe how each operator works.
To illustrate the effects of each operator, a visual example is provided afterwards.

``RSTREAM`` operator
^^^^^^^^^^^^^^^^^^^^

When ``RSTREAM`` is specified, all tuples in the relation are emitted.
In particular, a combination of ``RSTREAM`` with a ``RANGE 1 TUPLES`` window operator leads to 1:1 input/output behavior and can be processed by a faster execution plan than general statements.

In contrast,

::

    SELECT RSTREAM * FROM src [RANGE 100 TUPLES];

emits (at most) 100 tuples for every tuple in ``src``.


``ISTREAM`` operator
^^^^^^^^^^^^^^^^^^^^

When ``ISTREAM`` is specified, all tuples in the relation *that have not been in the previous relation* are emitted.
(The "I" in ``ISTREAM`` stands for "insert".)
Here, "previous" refers to the relation that was computed for the tuple just before the current tuple.
Therefore the current relation can contain at most one row that was not in the previous relation and thus ``ISTREAM`` can emit at most one row in each run.

In section 4.3.2 of [streamsql]_, it is highlighted that for the "is contained in previous relation" check, a notion of equality is required; in particular there are various possibilities how to deal with multiple tuples that have the same value.
In BQL tuples with the same value are considered equal, so that if the previous relation contains the values :math:`\{a, b\}` and the current relation contains the values :math:`\{b, a\}`, then nothing is emitted.
However, multiplicities are respected, so that if the previous relation contains the values :math:`\{b, a, b, a\}` and the current relation contains :math:`\{a, b, a, a\}`, then one :math:`a` is emitted.

As an example for a typical use case,

::

     SELECT ISTREAM * FROM src [RANGE 1 TUPLES];

will drop subsequent duplicates, i.e., emit only the first occurence of a series of tuples with identical values.

To illustrate the multiplicity counting,

::

    SELECT ISTREAM 1 FROM src [RANGE 3 TUPLES];

will emit three times :math:`1` and then nothing (because after the first three tuples processed, both the previous and the current relation always look like :math:`\{1, 1, 1\}`.)


``DSTREAM`` operator
^^^^^^^^^^^^^^^^^^^^

The ``DSTREAM`` operator is very similar to ``ISTREAM``, except that it emits all tuples in the *previous* relation that are not also contained in the current relation.
(The "D" in ``DSTREAM`` stands for "delete".)
Just as ``ISTREAM``, equality is computed using value comparison and multiplicity counting is used:
If the previous relation contains the values :math:`\{a, a, b, a\}` and the current relation contains :math:`\{b, b, a, a\}`, then one :math:`a` is emitted.

As an example for a typical use case,

::

     SELECT DSTREAM * FROM src [RANGE 1 TUPLES];

will emit only the last occurence of a series of tuples with identical values.

To illustrate the multiplicity counting,

::

    SELECT DSTREAM 1 FROM src [RANGE 3 TUPLES];

will never emit anything.


Examples
^^^^^^^^

To illustrate the difference between the three emit operators, a concrete example shall be presented.
The leftmost column shows the data of the tuple in the stream, next to that is the contents of the current window :math:`R(t^*)`, then the results of the relation-to-relation operator :math:`O(R(t^*))`, and finally a list of items that would be output by the respective emit operator.

Consider the following statement (where ``*STREAM`` is a placeholder for one of the emit operators)::

    SELECT *STREAM id, price FROM stream [RANGE 3 TUPLES] WHERE cat = 'toy';

This statement just takes the ``id`` and ``price`` key-value pairs of every tuple and outputs them untransformed.
The table below shows the emitted data for each emit operator.

.. |br| raw:: html

   <br />

+-------------------------------------------+------------------------------------------------+----------------------------------+----------------------------------+-----------------------------+-----------------------------+
| Current Tuple's Data                      | Current Window                                 | Output Relation                  | Emit Operator                                                                                |
+-------------------------------------------+------------------------------------------------+----------------------------------+----------------------------------+-----------------------------+-----------------------------+
|                                           | (last three tuples)                            |                                  | RSTREAM                          | ISTREAM                     | DSTREAM                     |
+===========================================+================================================+==================================+==================================+=============================+=============================+
| ``{"id": 1, "price": 3.5, "cat": "toy"}`` | ``{"id": 1, "price": 3.5, "cat": "toy"}``      | ``{"id": 1, "price": 3.5}``      | ``{"id": 1, "price": 3.5}``      | ``{"id": 1, "price": 3.5}`` |                             |
+-------------------------------------------+------------------------------------------------+----------------------------------+----------------------------------+-----------------------------+-----------------------------+
| ``{"id": 2, "price": 4.5, "cat": "toy"}`` | ``{"id": 1, "price": 3.5, "cat": "toy"}`` |br| | ``{"id": 1, "price": 3.5}`` |br| | ``{"id": 1, "price": 3.5}`` |br| |                             |                             |
|                                           | ``{"id": 2, "price": 4.5, "cat": "toy"}``      | ``{"id": 2, "price": 4.5}``      | ``{"id": 2, "price": 4.5}``      | ``{"id": 2, "price": 4.5}`` |                             |
+-------------------------------------------+------------------------------------------------+----------------------------------+----------------------------------+-----------------------------+-----------------------------+
| ``{"id": 3, "price": 10.5, "cat": "cd"}`` | ``{"id": 1, "price": 3.5, "cat": "toy"}`` |br| | ``{"id": 1, "price": 3.5}`` |br| | ``{"id": 1, "price": 3.5}`` |br| |                             |                             |
|                                           | ``{"id": 2, "price": 4.5, "cat": "toy"}`` |br| | ``{"id": 2, "price": 4.5}``      | ``{"id": 2, "price": 4.5}``      |                             |                             |
|                                           | ``{"id": 3, "price": 10.5, "cat": "cd"}``      |                                  |                                  |                             |                             |
+-------------------------------------------+------------------------------------------------+----------------------------------+----------------------------------+-----------------------------+-----------------------------+
| ``{"id": 4, "price": 8.5, "cat": "dvd"}`` | ``{"id": 2, "price": 4.5, "cat": "toy"}`` |br| | ``{"id": 2, "price": 4.5}``      | ``{"id": 2, "price": 4.5}``      |                             | ``{"id": 1, "price": 3.5}`` |
|                                           | ``{"id": 3, "price": 10.5, "cat": "cd"}`` |br| |                                  |                                  |                             |                             |
|                                           | ``{"id": 4, "price": 8.5, "cat": "dvd"}``      |                                  |                                  |                             |                             |
+-------------------------------------------+------------------------------------------------+----------------------------------+----------------------------------+-----------------------------+-----------------------------+
| ``{"id": 5, "price": 6.5, "cat": "toy"}`` | ``{"id": 3, "price": 10.5, "cat": "cd"}`` |br| |                                  |                                  |                             | ``{"id": 2, "price": 4.5}`` |
|                                           | ``{"id": 4, "price": 8.5, "cat": "dvd"}`` |br| |                                  |                                  |                             |                             |
|                                           | ``{"id": 5, "price": 6.5, "cat": "toy"}``      | ``{"id": 5, "price": 6.5}``      | ``{"id": 5, "price": 6.5}``      | ``{"id": 5, "price": 6.5}`` |                             |
+-------------------------------------------+------------------------------------------------+----------------------------------+----------------------------------+-----------------------------+-----------------------------+


.. [cql] Arasu et al., "The CQL Continuous Query Language: Semantic Foundations and Query Execution", http://ilpubs.stanford.edu:8090/758/1/2003-67.pdf

.. [streamsql] Jain et al., "Towards a Streaming SQL Standard", http://cs.brown.edu/~ugur/streamsql.pdf


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
