.. _ref_stmts_create_stream:

CREATE STREAM
=============

Synopsis
--------

::

    CREATE STREAM name AS select

Description
-----------

``CREATE STREAM`` creates a new stream (a.k.a a continuous view) in a topology.

Parameters
----------

name
    The name of the stream to be created.

select
    The ``SELECT`` statement to generate a stream. **select** can be any
    ``SELECT`` statement including a statement using ``UNION ALL``.

Examples
--------

To create a stream named "strm"::

    CREATE STREAM strm AS SELECT RSTREAM * FROM src [RANGE 1 TUPLES];

To create a stream which merges all tuples from multiple streams::

    CREATE STREAM strm AS
        SELECT RSTREAM * FROM src1 [RANGE 1 TUPLES]
        UNION ALL
        SELECT RSTREAM * FROM src2 [RANGE 1 TUPLES];
