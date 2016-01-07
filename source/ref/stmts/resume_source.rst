.. _ref_stmts_resume_source:

RESUME SOURCE
=============

Synopsis
--------

::

    RESUME SOURCE name

Description
-----------

``RESUME SOURCE`` resumes a paused source so that the source can start to emit
tuples again. Executing ``RESUME SOURCE`` on a running source doesn't affect anything.

``RESUME SOURCE`` fails if the source doesn't support the statement.

Parameters
----------

name
    The name of the source to be resumed.

Examples
--------

A common use case of ``RESUME SOURCE`` to resume a source which is created by
``CREATE PAUSED SOURCE``.

::

    CREATE PAUSED SOURCE src TYPE some_source_type WITH ...parameters...;

    -- ... construct a topology connected to src ...

    RESUME SOURCE src;

By doing this, no tuple emitted from `src` will be lost.
