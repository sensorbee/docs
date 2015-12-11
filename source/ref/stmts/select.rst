.. _ref_stmts_select:

SELECT
======

Synopsis
--------

::

    SELECT emitter {* | expression [AS output_name]} [, ...]
        FROM from_item "[" RANGE range_number {TUPLES | SECONDS | MILLISECONDS} "]"
            [AS stream_alias] [, ...]
        [WHERE condition [, ...]]
        [GROUP BY expression [, ...]]
        [HAVING condition [, ...]]
        [UNION ALL select]

    where emitter is:

        {ISTREAM | DSTREAM | RSTREAM}
            ["[" {
                LIMIT emitter_limit |
                EVERY sample_count-{ST | ND | RD | TH} TUPLE} |
                EVERY sample_time {SECONDS | MILLISECONDS} |
                SAMPLE sampling_rate %
            } "]"]

Description
-----------

``SELECT`` retrieves tuples from one or more streams. The general processing of
``SELECT`` is as follows:

#. Each tuple emitted from a **from_item** is computed. If more than one element
   is specified in the ``FROM`` clause, they are cross-joined.
#. When the `WHERE` clause is specified, **condition** is evaluated for each set
   of tuples cross-joined in the ``FROM`` clause. Sets which do not satisfy the
   condition are eliminated from the output.
#. TODO: window
#. TODO: ``GROUP BY`` and ``HAVING``
#. TODO: projection
#. TODO: emitter (ISTREAM/DSTREAM/RSTREAM)
#. TODO: ``UNION ALL``

Parameters
----------

emitter
^^^^^^^

``FROM`` Clause
^^^^^^^^^^^^^^^

``WHERE`` Clause
^^^^^^^^^^^^^^^^

``GROUP BY`` Clause
^^^^^^^^^^^^^^^^^^^

``HAVING`` Clause
^^^^^^^^^^^^^^^^^


Notes
-----

Evaluation in ``WHERE`` clause
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Each set of tuples cross-joined in the ``FROM`` clause is evaluated exactly once
in the ``WHERE`` clause. Therefore, all functions in the ``WHERE`` clause are
only called once for each set::

    SELECT RSTREAM * FROM stream1 [RANGE 100 TUPLES], stream2 [RANGE 100 TUPLES]
        WHERE random() < 0.2;

In this example, 80% of sets of cross-joined tuples are filtered out and only
20% of sets (around 20 tuples for each input from either stream) are emitted.
