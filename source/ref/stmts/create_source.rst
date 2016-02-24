.. _ref_stmts_create_source:

CREATE SOURCE
=============

Synopsis
--------

::

    CREATE [PAUSED] SOURCE name TYPE type_name [WITH parameter_name = parameter_value [, ...]]

Description
-----------

``CREATE SOURCE`` creates a new source in a topology.

Parameters
----------

``PAUSED``
    The source is paused when it's created if this option is used.

name
    The name of the source to be created.

type_name
    The type name of the source.

parameter_name
    The name of a source-specific parameter.

parameter_value
    The value for a source-specific parameter.

Source Parameters
-----------------

The optional ``WITH`` clause specifies parameters specific to the source.
See each source's documentation to find out parameters it provides.

Notes
-----

Some sources stop after emitting all tuples. They can stop even before any
subsequent statement is executed. For such sources, specify ``PAUSED`` parameter
and run :ref:`ref_stmts_resume_source` after completely setting up a topology so
that no tuple emitted from the source will be lost.

Examples
--------

To create a source having the name "src" with no source-specific parameter::

    CREATE SOURCE src TYPE dropped_tuples;

To create a source with source-specific parameters::

    CREATE SOURCE fluentd TYPE fluentd WITH bind = "0.0.0.0:12345",
      tag_field = "my_tag";

As you can see, the name of a source can be same as the type name of a source.
