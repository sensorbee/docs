*******
Queries
*******

The previous chapters described how to define data sources and sinks to communicate with the outside world.
Now it is discussed how to transform the data stream from those sources and write it to the defined sinks.

Processing Model
================

Overview
--------

The processing model in BQL is similar to what is explained in [cql]_.
In this model, each tuple in a stream has the shape :math:`(t, d)`, where :math:`t` is the original timestamp and :math:`d` the data contained.

In order to execute SQL-like queries, a finite set of tuples from the possibly unbounded stream, a *relation*, is required.
In the processing step at time :math:`t^*`, a *stream-to-relation* operator :math:`R` that converts a certain set of tuples in the stream to a relation :math:`R(t^*)` is used.
This relation is then processed with a *relation-to-relation* operator :math:`O` that is expressed in a form very closely related to an SQL ``SELECT`` statement.
Finally, a *relation-to-stream* operator :math:`S` will emit certain rows from the output relation :math:`O(R(t^*))` into the output stream, possibly taking into account the results of the previous execution step :math:`O(R(t^*_{\text{prev}}))`.

This three-step pipeline is executed for each tuple, but only for one tuple at a time.
Therefore, during execution there is a well-defined "current tuple".
This also means that if there is no tuple in the input stream for a long time, transformation functions will not be called.

Now the kind of stream-to-relation and relation-to-stream operators that can be used in BQL is explained.


Stream-to-Relation Operators
----------------------------

In BQL, there are two different stream-to-relation operators, a time-based one and a tuple-based one.
They are also called "window operators", since they define a sliding window on the input stream.
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
They are also called "emit operators", since they control how tuples are emitted as output.
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


Selecting and Transforming Data
===============================

In the previous section, it was explained how BQL converts stream data into relations and back.
This section is about how this relational data can be selected and transformed.
This functionality is exactly what SQL's ``SELECT`` statement was designed to do, and so in BQL the ``SELECT`` syntax is mimicked as much as possible.
(Some basic knowledge of what the SQL ``SELECT`` statement does is assumed.)
However, as opposed to the SQL data model, BQL's input data is assumed to be JSON-like, i.e., with varying shapes, nesting levels, and data types;
therefore the BQL ``SELECT`` statement has a number of small difference to SQL ``SELECT``.


Overview
--------

The general syntax of the ``SELECT`` command is

::

    SELECT emit_operator select_list FROM table_expression

The ``emit_operator`` is one of the operators described in `Relation-to-Stream Operators`_.
The following subsections describe the details of ``select_list`` and ``table_expression``.


Table Expressions
-----------------

A *table expression* computes a table.
The table expression contains a ``FROM`` clause that is optionally followed by ``WHERE``, ``GROUP BY``, and ``HAVING`` clauses::

    ... FROM table_list [WHERE filter_expression]
        [GROUP BY group_list] [HAVING having_expression]


The ``FROM`` Clause
^^^^^^^^^^^^^^^^^^^

The ``FROM`` clause derives a table from one or more other tables given in a comma-separated table reference list.

::

    FROM table_reference [, table_reference [, ...]]

In SQL, each ``table_reference`` is (in the simplest possible case) an identifier that refers to a pre-defined table, e.g., ``FROM users`` or ``FROM names, addresses, cities`` are valid SQL ``FROM`` clauses.

In BQL, only streams have identifiers, so in order to get a well-defined relation, a window specifier as explained in `Stream-to-Relation Operators`_ must be added.
In particular, the examples just given for SQL ``FROM`` clauses are all *not* valid in BQL, but the following are::

    FROM users [RANGE 10 TUPLES]

    FROM names [RANGE 2 TUPLES], addresses [RANGE 1.5 SECONDS], cities [RANGE 200 MILLISECONDS]


Using Stream Functions
""""""""""""""""""""""

BQL also knows "user-defined stream functions" (UDSF) that transform a stream into another stream and can be used, for example, to output multiple output rows per input row; something that is not possible with standard ``SELECT`` features.
(These are similar to "Table Functions" in PostgreSQL.)
Such UDSFs can also be used in the ``FROM`` clause:
Instead of using a stream's identifier, use the function call syntax ``function(param, param, ...)`` with the UDSF name as the function name and the base stream's identifier as its first parameter (as a string, i.e., in single quotes), possibly followed by other parameters.
For example, if there is a UDSF called ``duplicate`` that takes the input stream's name as first parameter (as all UDSFs do) and the number of copies of each input tuple as the second, this would look as follows::

    FROM duplicate('products', 3) [RANGE 10 SECONDS]


Table Joins
"""""""""""

If more than one table reference is listed in the ``FROM`` clause, the tables are cross-joined (that is, the Cartesian product of their rows is formed).
The syntax ``table1 JOIN table2 ON (...)`` is not supported in BQL.
The result of the ``FROM`` list is an intermediate virtual table that can then be subject to transformations by the ``WHERE``, ``GROUP BY``, and ``HAVING`` clauses and is finally the result of the overall table expression.


Table Aliases
"""""""""""""

A temporary name can be given to tables and complex table references to be used for references to the derived table in the rest of the query.
This is called a "table alias".
To create a table alias, write

::

    FROM table_reference AS alias

The use of table aliases is optional, but helps to shorten statements.
By default, each table can be addressed using the stream name or the UDSF name, respectively.
Therefore, table aliases are only mandatory if the same stream/UDSF is used multiple times in a join.
Taking aliases into account, each name must uniquely refer to one table. ``FROM stream [RANGE 1 TUPLES], stream [RANGE 2 TUPLES]`` or ``FROM streamA [RANGE 1 TUPLES], streamB [RANGE 2 TUPLES] AS streamA`` are not valid, but ``FROM stream [RANGE 1 TUPLES] AS streamA, stream [RANGE 2 TUPLES] AS streamB`` and also ``FROM stream [RANGE 1 TUPLES], stream [RANGE 2 TUPLES] AS other`` are.


The ``WHERE`` Clause
^^^^^^^^^^^^^^^^^^^^

The syntax of the ``WHERE`` clause is

::

    WHERE filter_expression

where ``filter_expression`` is any value expression that can be converted to boolean.
(That is, ``WHERE 6`` is also a valid filter.)

After the processing of the ``FROM`` clause is done, each row of the derived virtual table is checked against the search condition.
If the result of the condition is true, the row is kept in the output table, otherwise (i.e., if the result is false or null) it is discarded.
The search condition typically references at least one column of the table generated in the ``FROM`` clause; this is not required, but otherwise the ``WHERE`` clause will be fairly useless.

As BQL does not support the ``table1 JOIN table2 ON (condition)`` syntax, the join condition must always be given in the ``WHERE`` clause.


The ``GROUP BY`` and ``HAVING`` Clauses
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

After passing the ``WHERE`` filter, the derived input table might be subject to grouping, using the ``GROUP BY`` clause, and elimination of group rows using the ``HAVING`` clause.
They basically have the same semantics as explained in the `PostgreSQL Documentation, section 7.2.3 <http://www.postgresql.org/docs/9.5/static/queries-table-expressions.html#QUERIES-GROUP>`_

One current limitation of BQL row grouping is that only simple columns can be used in the ``GROUP BY`` list, no complex expressions are allowed.
For example, ``GROUP BY round(age/10)`` cannot be used in BQL at the moment.


Select Lists
------------

As shown in the previous section, the table expression in the SELECT command constructs an intermediate virtual table by possibly combining tables, views, eliminating rows, grouping, etc.
This table is finally passed on to processing by the "select list".
The select list determines which elements of the intermediate table are actually output.


Select-List Items
^^^^^^^^^^^^^^^^^

As in SQL, the select list contains a number of comma-separated expressions::

    SELECT emit_operator expression [, expression] [...] FROM ...

In general, items of a select list can be arbitrary `Value Expressions`_.
In SQL, tables are strictly organized in "rows" and "columns" and the most important elements in such expressions are therefore column references.

In BQL, each input tuple can be considered a "row", but the data can also be unstructured and the notion of a "column" is not sufficient.
Therefore, BQL uses `JSON Path <http://goessner.net/articles/JsonPath/>`_ to address data in each row.
If only one table is used in the ``FROM`` clause and only top-level keys of each JSON-like row are referenced, the BQL select list looks the same as in SQL::

    SELECT RSTREAM a, b, c FROM input [RANGE 1 TUPLES];

If the input data has the form ``{"a": 7, "b": "hello", "c": false}``, then the output will look exactly the same.
However, JSON Path allows to access nested elements as well::

    SELECT RSTREAM a.foo.bar FROM input [RANGE 1 TUPLES];

If the input data has the form ``{"a": {"foo": {"bar": 7}}}``, then the output will be ``{"col_1": 7}``.
(See paragraph `Column Labels`_ below for details on output key naming, and the section `Field Selectors`_ for details about the available syntax for JSON Path expressions.)


Table Prefixes
""""""""""""""

Where SQL uses the dot in ``SELECT left.a, right.b`` to specify the table from which to use a column, JSON Path uses the dot to describe a child relation in a single JSON element as shown above.
Therefore to avoid ambiguity, BQL uses the colon (``:``) character to separate table and JSON path::

    SELECT RSTREAM left:foo.bar, right:hoge FROM ...

If there is just one table to select from, the table prefix can be omitted, but then it must be omitted in *all* expressions of the statement.


Column Labels
^^^^^^^^^^^^^

The result value of every expression in the select list will be assigned to a key in the output row.
If not explicitly specified, these output keys will be ``"col_1"``, ``"col_2"``, etc. in the order the expressions were specified in the select list.
However, in some cases a more meaningful output key is chosen by default, as already shown above:

- If the expression is a single top-level key (like ``a``), then the output key will be the same.
- If the expression is a simple function call (like ``f(a)``), then the output key will be the function name.
- If the expression refers the timestamp of a tuple in a stream (using the ``stream:ts()`` syntax), then the output key will be ``ts``.
- If the expression is the wildcard (``*``), then the input will be copied, i.e., all keys from the input document will be present in the output document.

The output key can be overridden by specifying an ``... AS output_key`` clause after an expression.
For the example above,

::

    SELECT RSTREAM a.foo.bar AS x FROM input [RANGE 1 TUPLES];

will result in an output document that has the shape ``{"x": 7}`` instead of ``{"col_1": 7}``.
Note that it is possible to use the same column alias multiple times, but in this case it is undefined which of the values with the same alias will end up in that output key.

TODO: Explain `... AS foo.bar[4]` syntax.


Notes on Wildcards
^^^^^^^^^^^^^^^^^^

In SQL, the wildcard (``*``) can be used as a shorthand expression for all columns of an input table.
However, due to the strong typing in SQL's data model, name and type conflicts can still be checked at the time the statement is analyzed.
In BQL's data model, there is no strong typing, therefore the wildcard operator must be used with a bit of caution.
For example, in

::

    SELECT RSTREAM * FROM left [RANGE 1 TUPLES], right [RANGE 1 TUPLES];

if the data in the ``left`` stream looks like ``{"a": 1, "b": 2}`` and the data in the ``right`` stream looks like ``{"b": 3, "c": 4}``, then the output document will have the keys ``a``, ``b``, and ``c``, but the value of the ``b`` key is undefined.

To select all keys from only one stream, the colon notation (``stream:*``) as introduced above can be used.

The wildcard can be used with a column alias as well.
The expression ``* AS foo`` will nest the input document under the given key ``foo``, i.e., input ``{"a": 1, "b": 2}`` is transformed to ``{"foo": {"a": 1, "b": 2}}``.

On the other hand, it is also possible to use the wildcard as an alias, as in ``foo AS *``.
This will have the opposite effect, i.e., it takes the contents of the ``foo`` key (which *must* be a map itself) and pulls them up to top level, i.e., ``{"foo": {"a": 1, "b": 2}}`` is transformed to ``{"a": 1, "b": 2}``.

Note that any name conflicts that arise due to the use of the wildcard operator (e.g., in ``*``, ``a:*, b:*``, ``foo AS *, bar AS *``) lead to undefined values in the column with the conflicting name.
However, if there is an explicitly specified output key, this will always be prioritized over a key originating from a wildcard expression.


Examples
^^^^^^^^

Single Input Stream
"""""""""""""""""""

+-------------------+-----------------------+--------------------------+
| Select List       | Input Row             | Output Row               |
+===================+=======================+==========================+
| ``a``             | ``{"a": 1, "b": 2}``  | ``{"a": 1}``             |
+-------------------+-----------------------+--------------------------+
| ``a, b``          | ``{"a": 1, "b": 2}``  | ``{"a": 1, "b": 2}``     |
+-------------------+-----------------------+--------------------------+
| ``a + b``         | ``{"a": 1, "b": 2}``  | ``{"col_1": 3}``         |
+-------------------+-----------------------+--------------------------+
| ``a, a + b``      | ``{"a": 1, "b": 2}``  | ``{"a": 1, "col_2": 3}`` |
+-------------------+-----------------------+--------------------------+
| ``*``             | ``{"a": 1, "b": 2}``  | ``{"a": 1, "b": 2}``     |
+-------------------+-----------------------+--------------------------+


Join on Two Streams ``l`` and ``r``
"""""""""""""""""""""""""""""""""""

+-------------------+-----------------------+-----------------------+--------------------------------------+
| Select List       | Input Row (``l``)     | Input Row (``r``)     | Output Row                           |
+===================+=======================+=======================+======================================+
| ``l:a``           | ``{"a": 1, "b": 2}``  | ``{"c": 3, "d": 4}``  | ``{"a": 1}``                         |
+-------------------+-----------------------+-----------------------+--------------------------------------+
| ``l:a, r:c``      | ``{"a": 1, "b": 2}``  | ``{"c": 3, "d": 4}``  | ``{"a": 1, "c": 3}``                 |
+-------------------+-----------------------+-----------------------+--------------------------------------+
| ``l:a + r:c``     | ``{"a": 1, "b": 2}``  | ``{"c": 3, "d": 4}``  | ``{"col_1": 4}``                     |
+-------------------+-----------------------+-----------------------+--------------------------------------+
| ``l:*``           | ``{"a": 1, "b": 2}``  | ``{"c": 3, "d": 4}``  | ``{"a": 1, "b": 2}``                 |
+-------------------+-----------------------+-----------------------+--------------------------------------+
| ``l:*, r:c AS b`` | ``{"a": 1, "b": 2}``  | ``{"c": 3, "d": 4}``  | ``{"a": 1, "b": 3}``                 |
+-------------------+-----------------------+-----------------------+--------------------------------------+
| ``l:*, r:*``      | ``{"a": 1, "b": 2}``  | ``{"c": 3, "d": 4}``  | ``{"a": 1, "b": 2, "c": 3, "d": 4}`` |
+-------------------+-----------------------+-----------------------+--------------------------------------+
| ``*``             | ``{"a": 1, "b": 2}``  | ``{"c": 3, "d": 4}``  | ``{"a": 1, "b": 2, "c": 3, "d": 4}`` |
+-------------------+-----------------------+-----------------------+--------------------------------------+
| ``*``             | ``{"a": 1, "b": 2}``  | ``{"b": 3, "d": 4}``  | ``{"a": 1, "b": (undef.), "d": 4}``  |
+-------------------+-----------------------+-----------------------+--------------------------------------+


Building Processing Pipelines
=============================

The ``SELECT`` statement as described above returns a data stream (where the transport mechanism depends on the client in use), but often an unattended processing pipeline (i.e., running on the server without client interaction) needs to set up.
In order to do so, a stream can be created from the results of a ``SELECT`` query and then used afterwards like an input stream.
(The concept is equivalent to that of an SQL ``VIEW``.)

The statement used to create a stream from an SELECT statement is::

    CREATE STREAM stream_name AS select_statement;

For example::

    CREATE STREAM odds AS SELECT RSTREAM * FROM numbers [RANGE 1 TUPLES] WHERE id % 2 = 1;

If that statement is issued correctly, subsequent statements can refer to ``stream_name`` in their ``FROM`` clauses.

If a stream thus created is no longer needed, it can be dropped using the ``DROP STREAM`` command::

    DROP STREAM stream_name;


Data Output
===========

To write data to a sink, there are two different statements.
If *all* data from a stream is to be written to a sink unaltered, the

::

    INSERT INTO sink_name FROM stream_name;

statement can be used.

If the stream that is used to insert into the sink was created only for that purpose, the ``CREATE STREAM`` and ``INSERT INTO`` statements can be merged to one::

    INSERT INTO sink_name select_statement;

For example::

    INSERT INTO notifier SELECT RSTREAM * FROM events [RANGE 1 TUPLES] WHERE importance > 5;


Expression Evaluation
=====================

To evaluate expressions outside the context of a stream, the ``EVAL`` command can be used.
The general syntax is

::

    EVAL expression;

and ``expression`` can generally be any expression, but it cannot contain references to any columns, aggregate functions or anything that only makes sense in a stream processing context.

For example, in the SensorBee Shell, the following can be done::

    >>> EVAL 'foo' || 'bar';
    foobar

