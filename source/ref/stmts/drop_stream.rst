.. _ref_stmts_drop_stream:

DROP STREAM
===========

Synopsis
--------

::

    DROP STREAM name

Description
-----------

``DROP STREAM`` drops a stream that is already created in a topology. The
stream can no longer be used after executing the statement.

Parameters
----------

name
    The name of the stream to be dropped.

Examples
--------

To drop a stream having the name "strm"::

    DROP STREAM strm;
