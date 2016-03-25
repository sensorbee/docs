.. _ref_stmts_insert_into:

INSERT INTO
===========

Synopsis
--------

::

    INSERT INTO sink FROM stream

Description
-----------

``INSERT INTO`` inserts tuples from a stream or a source to a sink.

Parameters
----------

sink
    The name of the sink to which tuples are inserted.

stream
    The name of a stream or a source.

Examples
--------

To insert tuples into a sink from a source having the name "src"::

    INSERT INTO snk FROM src;
