.. _ref_stmts_select:

SELECT
======

Synopsis
--------

::

    SELECT emitter {* | expression [AS output_name]} [, ...]
        FROM from_item stream_to_relation_operator
            [AS stream_alias] [, ...]
        [WHERE condition [, ...]]
        [GROUP BY expression [, ...]]
        [HAVING condition [, ...]]
        [UNION ALL select]

    where emitter is:

        {RSTREAM | ISTREAM | DSTREAM}

    where stream_to_relation_operator is:

        "[" RANGE range_number {TUPLES | SECONDS | MILLISECONDS}
            [, BUFFER SIZE buffer_size]
            [, drop_mode IF FULL]
        "]"

Description
-----------

``SELECT`` retrieves tuples from one or more streams. The general processing of
``SELECT`` is as follows:

#. Each **from_item** is converted into a relation (i.e. a window) from a
   stream. Then, each tuple emitted from a **from_item** is computed within
   the window. If more than one element is specified in the ``FROM`` clause,
   they are cross-joined.
#. When the `WHERE` clause is specified, **condition** is evaluated for each set
   of tuples cross-joined in the ``FROM`` clause. Sets which do not satisfy the
   condition are eliminated from the output.
#. If the ``GROUP BY`` clause is specified, tuples satisfied the condition in
   the ``WHERE`` clause are combined into groups based on the result of expressions. Then, aggregate functions are performed on each group. When
   the ``HAVING`` clause is given, it eliminates groups that do not satisfy
   the given condition.
#. The output tuples are computed using the ``SELECT`` output expressions for
   each tuple or group.
#. Computed output tuples are converted into a stream using
   :ref:`bql_queries_relation_to_stream_operators` and emitted from the
   ``SELECT``.
#. ``UNION ALL``, if present, combines outputs from multiple ``SELECT``. It
   simply emits all tuples from all ``SELECT`` without considering duplicates.

Parameters
----------

emitter
^^^^^^^

**emitter** controls how a ``SELECT`` emits resulting tuples.

``RSTREAM``
    When ``RSTREAM`` is specified, all tuples in a relation as a result of
    processing a newly coming tuple are output. See
    :ref:`bql_queries_relation_to_stream_operators` for more details.

``ISTREAM``
    When  ``ISTREAM`` is specified, tuples contained in the current relation
    but not in the previously computed relation are emitted. In other words,
    tuples that are newly inserted or updated since the previous computation
    are output. See :ref:`bql_queries_relation_to_stream_operators` for more details.

``DSTREAM``
    When ``DSTREAM`` is specified, tuples contained in the previously computed
    relation but not in the current relation are emitted. In other words,
    tuples in the previous relation that are deleted or updated in the current
    relation are output. Note that output tuples are from the previous
    relation so that they have old values. See
    :ref:`bql_queries_relation_to_stream_operators` for more details.

..
    The following parameters are intentionally undocumented at the moment
    because their specification related to computational model would likely
    be changed soon.
    ["[" {
        LIMIT emitter_limit |
        EVERY sample_count-{ST | ND | RD | TH} TUPLE} |
        EVERY sample_time {SECONDS | MILLISECONDS} |
        SAMPLE sampling_rate %
    } "]"]

``FROM`` clause
^^^^^^^^^^^^^^^

The ``FROM`` clause specifies one or more source streams for the ``SELECT``
and converts those streams into relations using stream to relation operators.

The ``FROM`` clause contains following parameters:

from_item
    **from_item** is a source stream which can be either a source, a stream,
    or a UDSF.

range_number
    **range_number** is a numeric value which specifies how many tuples are in the
    window. **range_number** is followed by one of interval types:
    ``TUPLES``, ``SECONDS``, or ``MILLISECONDS``.

    When ``TUPLES`` is given, the window can contain at most **range_number**
    tuples. If a new tuple is inserted into the window having **range_number**
    tuples, the oldest tuple is removed. "The oldest tuple" is the tuple that
    was inserted into the window before any other tuples, not the tuple having
    the oldest timestamp.

    When ``SECONDS`` or ``MILLISECONDS`` is specified, the difference of the
    minimum and maximum timestamps of tuples in the window can be at most
    **range_number** seconds or milliseconds. If a new tuple is inserted into
    the window, tuples whose timestamp is **range_number** seconds or
    milliseconds earlier than the new tuple's timestamp are removed.

buffer_size
    **buffer_size** specifies the size of buffer, or a queue, located between
    **from_item** and the ``SELECT``. **buffer_size** must be an integer and
    greater than 0.

    .. todo:: define the maximum buffer size

drop_mode
    **drop_mode** controls how a new tuple is inserted into the buffer located
    between **from_item** and the ``SELECT`` when the buffer is full.
    **drop_mode** can be one of the followings:

    * ``WAIT``

        * A new tuple emitted from **from_item** is blocked until the
          ``SELECT`` consumes at least one tuple.

    * ``DROP OLDEST``

        * The oldest tuple in the buffer is removed and a new tuple is
          inserted into the buffer. "The oldest tuple" is the tuple that was
          inserted into the buffer before any other tuples, not the tuple
          having the oldest timestamp.

    * ``DROP NEWEST``

        * The oldest tuple in the buffer is removed and a new tuple is
          inserted into the buffer. "The newest tuple" is the tuple that was
          inserted into the buffer after any other tuples, not the tuple
          having the newest timestamp.

    .. todo:: describe the difference between a buffer and a window.

**stream_alias**
    **stream_alias** provides an alias of **from_item** and it can be referred
    by the alias in other parts of the ``SELECT``. If the alias is given, the
    original name is hidden and cannot be used to refer **from_item**.

``WHERE`` clause
^^^^^^^^^^^^^^^^

The ``SELECT`` can optionally have a ``WHERE`` clause. The ``WHERE`` clause
have a condition. The condition can be any expression that evaluates to a
result of type ``bool``. Any tuple that does not satisfy the condition
(i.e. the result of the expression is ``false``) will be eliminated from the
output.

:ref:`bql_operators` describes operators that can be used in the condition.

``GROUP BY`` clause
^^^^^^^^^^^^^^^^^^^

The ``GROUP BY`` clause is an optional clause and condenses into a single
tuple all selected tuples whose expressions specified in ``GROUP BY`` clause
result in the same value.

**expression** can be any expression using fields of an input tuple. When
there're multiple expressions in the clause, tuples having the same set of
values computed from those expressions are grouped into a single tuple.

When the ``GROUP BY`` clause is present, any ungrouped field cannot be used
as an output field without aggregate functions. For example, when tuples have
4 fields ``a``, ``b``, ``c``, and ``d``, and the ``GROUP BY`` clause has
following expressions::

    GROUP BY a, b + c

``a`` can only be used as an output field::

    SELECT a FROM stream [RANGE 1 TUPLES]
    GROUP BY a, b + c;

Other fields need to be specified in aggregate functions::

    SELECT a, max(b), min(b + c), avg(c * d) FROM stream [RANGE 1 TUPLES]
    GROUP BY a, b + c;

Aggregate functions are evaluated for each group using all tuples in the
group.


.. note::

    The ``GROUP BY`` clause performs grouping within a window::

        SELECT a FROM stream [RANGE 10 TUPLES]
        GROUP BY a;

    This ``SELECT`` computes at most 10 groups of tuples because there're only
    10 tuples in the window.

``HAVING`` clause
^^^^^^^^^^^^^^^^^

The ``HAVING`` clause is an optional clause and placed after the ``GROUP BY``
clause. The ``HAVING`` clause has an condition and evaluate it for each group,
instead of each tuple. When ungrouped fields are used in the condition, they
need to be in aggregate functions::

    SELECT a, max(b), min(b + c), avg(c * d) FROM stream [RANGE 1 TUPLES]
    GROUP BY a, b + c HAVING min(b + c) > 1 AND avg(c * d) < 10;

In this example, ``b``, ``c``, and ``d`` are ungrouped fields and cannot
directly specified in the condition.

``SELECT`` list
^^^^^^^^^^^^^^^

Notes
-----

An emitter and its performance
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

There're some use case specific optimizations of the evaluation of the
``SELECT`` and this subsection describes each optimization and its limitation.

Simple transformation and filtering
"""""""""""""""""""""""""""""""""""

Performing a simple per-tuple transformation or filtering over an input
stream is a very common task. Therefore, BQL optimizes statements having the
following form::

    SELECT RSTREAM projection FROM input [RANGE 1 TUPLES] WHERE conditions;

Limitations of this optimization are:

* There can only be one input stream and its range is ``[RANGE 1 TUPLES]``.
* The emitter must be ``RSTREAM``.

Evaluation in ``WHERE`` clause
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Each set of tuples cross-joined in the ``FROM`` clause is evaluated exactly once
in the ``WHERE`` clause. Therefore, all functions in the ``WHERE`` clause are
only called once for each set::

    SELECT RSTREAM * FROM stream1 [RANGE 100 TUPLES], stream2 [RANGE 100 TUPLES]
        WHERE random() < 0.2;

In this example, 80% of sets of cross-joined tuples are filtered out and only
20% of sets (around 20 tuples for each input from either stream) are emitted.
