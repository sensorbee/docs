.. _ref_stmts_pause_source:

PAUSE SOURCE
============

Synopsis
--------

::

    PAUSE SOURCE name

Description
-----------

``PAUSE SOURCE`` pauses a running source so that the source stops emitting
tuples until executing :ref:`ref_stmts_resume_source` on it again. Executing
``PAUSE SOURCE`` on a paused source doesn't affect anything.

``PAUSE SOURCE`` fails if the source doesn't support the statement.

Parameters
----------

name
    The name of the source to be paused.

Examples
--------

To pause a source named "src"::

    PAUSE SOURCE src;
