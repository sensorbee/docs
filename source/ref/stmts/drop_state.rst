.. _ref_stmts_drop_state:

DROP STATE
==========

Synopsis
--------

::

    DROP STATE name

Description
-----------

``DROP STATE`` drops a UDS that is already created in a topology. The UDS can
no longer be used after executing the statement.

.. note::

    Even if a ``uds`` sink exist for the UDS, the sink will not be dropped when
    dropping the UDS. The ``uds`` sink must be dropped manually.

Parameters
----------

name
    The name of the UDS to be dropped.


Examples
--------

To drop a UDS named "my_uds"::

    DROP STATE my_ds;
