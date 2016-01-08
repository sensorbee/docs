.. _ref_stmts_save_state:

SAVE STATE
==========

Synopsis
--------

::

    SAVE STATE name [TAG tag]

Description
-----------

``SAVE STATE`` saves a UDS to SensorBee's storage. The location or the format
of the saved UDS depends on a storage that SensorBee uses and is not
controllable from BQL.

``SAVE STATE`` fails if the UDS doesn't support the statement.

Parameters
----------

name
    The name of the UDS to be saved.

tag
    The name of the user defined tag for versioning of the saved UDS data.
    When **tag** is omitted, "default" is used as the default tag name.

Examples
--------

To save a UDS named "my_uds" without a tag::

    SAVE STATE my_uds;

To save a UDS with a tag "trained"::

    SAVE STATE my_uds TAG trained;
