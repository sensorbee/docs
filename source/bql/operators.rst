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
    each unary minus operators must be separated by a space like ``- - -3``
    because ``--`` and succeeding characters are parsed as a comment. For
    example, ``---3`` is parsed as ``--`` and a comment body ``-3``.

String operators
================

BQL provides following string operators:

.. csv-table::
   :header: "Operator", "Description", "Example", "Result"

   ``||``, Concatenation, "``'Hello' || ', world'``", "``'Hello, world'``"

All types convertible to a string can be an operand of ``||``. When an operand
isn't a string, it's converted to string. Then, it's concatenated as a string.
For example, ``'a' || 1`` results in ``'a1'`` and ``10 || true`` will be
``10true``.

Comparison operators
====================

BQL provides following comparison operators:

.. csv-table::
   :header: "Operator", "Description", "Example", "Result"

   ``<``, Less than, ``1 < 2``, ``true``
   ``>``, Greater than, ``1 > 2``, ``false``
   ``<=``, Less than or equal to, ``1 <= 2``, ``true``
   ``>=``, Greater than or equal to, ``1 >= 2``, ``false``
   ``=``, Equal to, ``1 = 2``, ``false``
   ``<>`` or ``!=``, Not equal to, ``1 != 2``, ``true``
   ``IS NULL``, Null check, ``false IS NULL``, ``false``
   ``IS NOT NULL``, Non-null check, ``false IS NOT NULL``, ``true``

All comparison operators return a boolean value.

``<``, ``>``, ``<=``, and ``>=`` are only valid when both operands are numeric
values (i.e. integers or floating point numbers) or have the same type that is
comparable. Following types are comparable:

* ``null``
* ``int``
* ``float``
* ``string``
* ``timestamp``

Valid examples are as follows:

* ``1 < 2.1``

    * Integers and floating point numbers can be compared.

* ``'abc' > 'def'``
* ``1::timestamp <= 2::timestamp``
* ``NULL > 'a'``
    * This expression is valid although it always results in ``NULL``. See
      :ref:`bql_operators_null_comparison` below.

``=``, ``<>``, and ``!=`` are valid for any type even if both operands have
different types. When the types of operands are different, ``=`` results in
``false`` and others return ``true``. However, integers and floating point
numbers can be compared. For example ``1 = 1.0`` returns ``true``. When
operands have the same type, ``=`` results in ``true`` if both values are
equivalent and others return ``false``.

There's one exception. When one operand of comparison operators except
``IS NULL`` is a floating point value and its value is infinite, they return
``false``.

.. _bql_operators_null_comparison:

``NULL`` comparison
-------------------

In a three-valued logic, comparing any value with ``NULL`` results in ``NULL``.
For example, all of following expressions result in ``NULL``:

* ``1 < NULL``
* ``2 > NULL``
* ``"a" <= NULL``
* ``3 = NULL``
* ``NULL = NULL``
* ``NULL <> NULL``

Therefore, do **not** look for ``NULL`` values with ``expression = NULL``.
To check if a value is ``NULL`` or not, use ``IS NULL`` or ``IS NOT NULL``
operator. ``expression IS NULL`` operator returns ``true`` only when an
expression is ``NULL``.

.. note::

    ``[NULL] = [NULL]`` and ``{'a': NULL} = {'a': NULL}`` result in ``true``
    although it contradict the three-valued logic. This specification is
    provided for convenience. Arrays or maps often have ``NULL`` to indicate
    that there's no value for a specific key but the key actually exists. In
    other words, ``{'a': NULL, 'b': 1}`` and ``{'b': 1}`` are different.
    Therefore, ``NULL`` in arrays are maps are compared as if it's a regular
    value. Unlike ``NULL``, comparing infinite floating point values
    always result in ``false``.

Logical operators
=================

BQL provides following logical operators:

.. csv-table::
   :header: "Operator", "Description", "Example", "Result"

   ``AND``, Logical and, ``1 < 2 AND 2 < 3``, ``true``
   ``OR``, Logical or, ``1 < 2 OR 2 > 3``, ``true``
   ``NOT``, Logical negation, ``NOT 1 < 2``, ``false``

Logical operators also follow the three-valued logic. For example,
``true AND NULL`` and ``NULL OR false`` result in ``NULL``.

JSON operators
==============

Operator precedence
===================
