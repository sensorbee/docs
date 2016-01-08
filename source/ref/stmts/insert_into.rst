.. _ref_stmts_insert_into:

INSERT INTO
===========

Synopsis
--------

::

    INSERT INTO sink {select | FROM stream}

Description
-----------

``INSERT INTO`` inserts tuples from a stream or a source to a sink.

Parameters
----------

sink
    The name of the sink to which tuples are inserted.

select
    A ``SELECT`` statement generating a stream of tuples to be inserted into
    the sink.

stream
    The name of a stream or a source.

Examples
--------

To insert tuples into a sink named "snk" using a ``SELECT``::

    INSERT INTO snk SELECT RSTREAM * FROM src [RANGE 1 TUPLES] WHERE price > 10000;

The ``SELECT`` can contain multiple streams or any condition.

To insert tuples into a sink from a source having the name "src"::

    INSERT INTO snk FROM src;

This is a short form of

::

    INSERT INTO snk SELECT RSTREAM * FROM src [RANGE 1 TUPLES];
