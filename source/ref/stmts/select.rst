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

        {RSTREAM | ISTREAM | DSTREAM}

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

**emitter** controls how a ``SELECT`` emits resulting tuples.

``RSTREAM``
    When ``RSTREAM`` is specified, all tuples in a relation as a result of
    processing a newly coming tuple are output. See :ref:`Relation-to-Stream operators`
    for more details.

``ISTREAM``
    When  ``ISTREAM`` is specified, tuples contained in the current relation
    but not in the previously computed relation are emitted. In other words,
    tuples that are newly inserted or updated since the previous computation
    are output. See :ref:`Relation-to-Stream operators` for more details.

``DSTREAM``
    When ``DSTREAM`` is specified, tuples contained in the previously computed
    relation but not in the current relation are emitted. In other words,
    tuples in the previous relation that are deleted or updated in the current
    relation are output. Note that output tuples are from the previous
    relation so that they have old values. See :ref:`Relation-to-Stream operators`
    for more details.

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

An emitter and its performance
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

There're some use case specific optimizations and this subsection describes
each optimization and its limitation.

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
