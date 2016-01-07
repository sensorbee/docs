.. _ref_stmts_drop_sink:

DROP SINK
=========

Synopsis
--------

::

    DROP SINK name

Description
-----------

``DROP SINK`` drops a sink that is already created in a topology. The sink can
no longer be used after executing the statement.

Parameters
----------

name
    The name of the sink to be dropped.

Examples
--------

To drop a sink having the name "snk"::

    DROP SINK snk;
