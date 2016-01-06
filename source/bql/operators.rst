.. _bql_operators:

*********
Operators
*********

This chapter introduces operators used in BQL.

Arithmetic operators
====================

BQL provides following arithmetic operators:

.. csv-table::
   :header: "Operator", "Description", "Example", "Result"

   ``+``, Addition, ``6 + 1``, ``7``
   ``-``, Subtraction, ``6 - 1``, ``5``
   ``+``, Unary plus, ``+4``, ``4``
   ``-``, Unary minus, ``-4``, ``-4``
   ``*``, Multiplication, ``3 * 2``, ``6``
   ``/``, Division, ``7 / 2``, ``3``
   ``%``, Modulo, ``5 % 3``, ``2``

All operators accept both integers and floating point numbers. Integers and
floating point numbers can be mixed in a single arithmetic expression. For
example, ``3 + 5 * 2.5`` is valid.

.. note::

    Unary minus operators can be applied to a value multiple times. However,
    each unary minus operators must be seperated by a space like ``- - -3``
    because ``--`` and succeeding characters are parsed as a comment. For
    example, ``---3`` is parsed as ``--`` and a comment body ``-3``.

String operators
================

Comparison operators
====================

Logical operators
=================

JSON operators
==============

Operator precedence
===================
