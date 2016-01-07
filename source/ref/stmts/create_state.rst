.. _ref_stmts_create_state:

CREATE STATE
============

Synopsis
--------

::
    CREATE STATE name TYPE type_name [WITH parameter_name = parameter_value [, ...]]

Description
-----------

``CREATE STATE`` creates a new UDS (User Defined State) in a topology.

Parameters
----------

name
    The name of the UDS to be created.

type_name
    The type name of the UDS.

parameter_name
    The name of a UDS-specific parameter.

parameter_value
    The value for a UDS-specific parameter.

UDS Parameters
--------------

The optional ``WITH`` clause specifies parameters specific to the UDS.
See each UDS's documentation to find out parameters it provides.

Examples
--------

To create a UDS named "my_uds" with no UDS-specific parameter::

    CREATE STATE my_uds TYPE my_uds_type;

To create a UDS with UDS-specific parameters::

    CREATE STATE my_ids TYPE snowflake_id WITH machine_id = 1;
