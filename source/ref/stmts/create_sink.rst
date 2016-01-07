.. _ref_stmts_create_sink:

CREATE SINK
===========

Synopsis
--------

::

    CREATE SINK name TYPE type_name [WITH parameter_name = parameter_value [, ...]]

Description
-----------

``CREATE SINK`` creates a new sink in a topology.

Parameters
----------

name
    The name of the sink to be created.

type_name
    The type name of the sink.

parameter_name
    The name of a sink-specific parameter.

parameter_value
    The value for a sink-specific parameter.

Sink Parameters
---------------

The optional ``WITH`` clause specifies parameters specific to the sink.
See each sink's documentation to find out parameters it provides.

Examples
--------

To create a sink having the name "snk" with no sink-specific parameter::

    CREATE SINK snk TYPE fluentd;

To create a sink with sink-specific parameters::

    CREATE SINK fluentd TYPE fluentd WITH tag_field = 'fluentd_tag';

As you can see, the name of a sink can be same as the type name of a sink.
