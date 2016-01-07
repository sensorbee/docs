.. _ref_stmts_drop_source:

DROP SOURCE
===========

Synopsis
--------

::

    DROP SOURCE name

Description
-----------

``DROP SOURCE`` drops a source that is already created in a topology. The
source is stopped and removed from a topology. After executing the statement,
the source cannot be used.

Parameters
----------

name
    The name of the source to be dropped.

Examples
--------

To drop a source having the name "src"::

    DROP SOURCE src;
