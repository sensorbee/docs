*******
Queries
*******

This chapter describes how to retrieve streaming data using BQL.

Overview
========

TODO: refine following sections later

TODO: Create a section "Converting relation<->stream" and put Stream-To-Relation operators and Relation-to-Stream operators into it.

Stream-to-Relation operators
============================

TODO: describe how Stream-to-Relation operators work

A relation is a finite set of tuples (just like a table in a RDBMS). The
resulting relation for a ``SELECT`` statement is computed as follows:

1. An input stream is converted to a relation using a window (e.g. ``RANGE``).
2. When there're multiple input streams, all tuples in relations converted
   from each input stream are cross-joined.
3. TODO: WHEN, GROUP BY, HAVING

Relation-to-Stream operators
============================

Once a resulting relation is computed, tuples in the relation needs to be
output as a stream again so that it can be referred by other ``SELECT``
statements as an input stream. To convert a relation to a stream, BQL provides
following three relation-to-stream operators:

* ``RSTREAM``
* ``ISTREAM``
* ``DSTREAM``

The following subsections describe how each operator works.

``RSTREAM`` operator
--------------------

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
--------------------

TODO: description


``DSTREAM`` operator
--------------------

TODO: description


