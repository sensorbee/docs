.. _ref_stmts_rewind_source:

REWIND SOURCE
=============

Synopsis
--------

::

    REWIND SOURCE name

Description
-----------

``REWIND SOURCE`` rewinds a source so that the source emits tuples from the
beginning again.

``REWIND SOURCE`` fails if the source doesn't support the statement.

Parameters
----------

name
    The name of the source to be rewound.
