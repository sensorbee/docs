.. _ref_stmts_load_state:

LOAD STATE
==========

Synopsis
--------

::

    LOAD STATE name TYPE type_name [TAG tag]
        [SET set_parameter_name = set_parameter_key]
        [create_if_not_saved]

    where create_if_not_saved is:

        OR CREATE IF NOT SAVED
            [WITH create_parameter_name = create_parameter_value]

Description
-----------

``LOAD STATE`` loads a UDS that is previously saved by
:ref:`ref_stmts_save_state`.

``LOAD STATE`` fails if the UDS hasn't been saved yet. When
``OR CREATE IF NOT SAVED`` is specified, ``LOAD STATE`` creates a new UDS with
the given optional parameters if the UDS hasn't been saved yet.

``LOAD STATE``, even with ``OR CREATE IF NOT SAVED``, fails if the UDS doesn't
support the statement.

Parameters
----------

name
    The name of the UDS to be loaded.

type_name
    The type name of the UDS.

tag
    The name of the user defined tag for versioning of the saved UDS data.
    When **tag** is omitted, "default" is used as the default tag name.

set_parameter_name
    The name of a UDS-specific parameter defied for ``LOAD STATE``.

set_parameter_value
    The value for a UDS-specific parameter defined for ``LOAD STATE``.

create_parameter_name
    The name of a UDS-specific parameter defined for ``CREATE STATE``.

create_parameter_value
    The value for a UDS-specific parameter defined for ``CREATE STATE``.

``LOAD STATE`` can have two sets of parameters: **set_parameters** and
**create_parameters**. **set_parameters** are used when there's a saved UDS
data having the given tag. On the other hand, **create_parameters** are used
when the UDS hasn't been saved yet. **create_parameters** are exactly same as
parameters that the UDS defines for :ref:`ref_stmts_create_state`. However,
**set_parameters** are often completely different from **create_parameters**.
Because **create_parameters** are often saved as a part of the UDS's
information by ``SAVE STATE``, **set_parameters** doesn't have to have the same
set of parameters defined in **create_parameters**.

There're some use-cases that a UDS uses **set_parameters**:

* Customize loading behavior

    * When a UDS doesn't provide a proper versioning of saved data,
      ``LOAD STATE`` may fail to load it due to the format incompatibility. In
      such a case, it's difficult to modify saved binary data to have a format
      version number. Thus, providing a **set_parameter** specifying the format
      version number could be the only solution.

* Overwrite some saved values of **create_parameters**

Like **create_parameters**, **set_parameters** are specific to each UDS. See
the documentation of each UDS to find out parameters it provides.

Examples
--------

To load a UDS named "my_uds" and having the type "my_uds_type"::

    LOAD STATE my_uds TYPE my_uds_type;

Note that "my_uds" needs to be saved before executing this statement. Otherwise, it fails.

To load a UDS named "my_uds" and having the type "my_uds_type" and assigned
a tag "trained"::

    LOAD STATE my_uds TYPE my_uds_type TAG trained;

To load a UDS with **set_parameters**::

    LOAD STATE my_uds TYPE my_uds_type TAG trained
        SET force_format_version = "v1";

To load a UDS that hasn't been saved yet with ``OR CREATE IF NOT SAVED``::

    LOAD STATE my_uds TYPE my_uds_type OR CREATE IF NOT SAVED;

To load a UDS that hasn't been saved yet with ``OR CREATE IF NOT SAVED`` with
**create_parameters**::

    LOAD STATE my_uds TYPE my_uds_type OR CREATE IF NOT SAVED
        WITH id = 1;

When the UDS hasn't been saved previously, the statement above falls back into
``CREATE STATE`` as follows::

    CREATE STATE my_uds TYPE my_uds_type WITH id = 1;

``OR CREATE IF NOT SAVED`` can be used with a tag and **set_parameters**::

    LOAD STATE my_uds TYPE my_uds_type TAG trained SET force_format_version = "v1"
        OR CREATE IF NOT SAVED WITH id = 1;
