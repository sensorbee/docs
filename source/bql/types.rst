**************************
Data Types and Conversions
**************************

This chapter describes data types defined in BQL and how their type conversion
works.

Overview
========

BQL has following data types:

.. csv-table::
    :header: "Type name", "Description", "Example"

    ``null``, Null type, ``null``
    ``bool``, Boolean, ``true``
    ``int``, 64-bit integer, ``12``
    ``float``, 64-bit floating point number, ``3.14``
    ``string``, String, ``'sensorbee'``
    ``blob``, Binary large object, A blob value cannot directly be written in BQL.
    ``timestamp``, Datetime information in UTC, A timestamp value cannot directly be written in BQL.
    ``array``, Array, "``[1, '2', 3.4]``"
    ``map``, Map with string keys, "``{'a': 1, 'b': '2', 'c': 3.4}``"

These types are designed to work well with JSON. They can be converted to or
from JSON with some restrictions.

.. note::

    User defined types are not availble at the moment.

Types
=====

This section describes the detailed sepcification of each type.

``null``
--------

The type ``null`` only has one value: ``NULL``, which represents an empty or
undefined value.

``array`` can contain ``NULL`` as follows::

    [1, NULL, 3.4]

``map`` can also contain ``NULL`` as its value::

    {
        'some_key': NULL
    }

This map is different from an empty map ``{}`` because the key ``'some_key'``
actually exists in the map but the empty map doesn't even have a key.

``NULL`` is converted to ``null`` in JSON.

``bool``
--------

The type ``bool`` has two values: ``true`` and ``false``. In terms of a
three-valued logic, ``NULL`` represents the third state, "unknown".

``true`` and ``false`` are converted to ``true`` and ``false`` in JSON,
respectively.

``int``
-------

The type ``int`` is a 64-bit integer type. Its minimum value is
``-9223372036854775808`` and its maximum value is ``+9223372036854775807``.
Using an integer value out of this range result in an error.

An ``int`` value is converted to a number in JSON.

.. note::

    Some implementations of JSON use 64-bit floating point number for all
    numerical values. Therefore, they might not be able to handline integers
    greater than or equal to 9007199254740992 (i.e. ``2^53``) accurately.

``float``
---------

The type ``float`` is a 64-bit floating point type. Its implementation is
IEEE 754 on most platforms but some platforms could use other implementations.

A ``float`` value is converted to a number in JSON.

.. note::

    Following syntaxes and values are not supported in BQL yet:

    * Scientific notations: ``1e+10``
    * Infinity
    * NaN

    However, some expressions and functions may result in an infinity or a NaN.
    Because JSON doesn't have an infinity or a NaN notation, they'll become
    ``NULL`` when they're converted to JSON.

``string``
----------

The type ``string`` is similar to SQL's type ``text``. It may contain an
arbitrary length of characters. The value needs to be enclosed with single
quotes: ``'string value'``. It may contain any valid UTF character including a
null character.

A ``string`` value is converted to a string in JSON.

.. note::

    An escape sequence like ``'\a'`` or ``'\x32'`` isn't supported in BQL's
    ``string`` yet.

``blob``
--------

The type ``blob`` is a data type for any variable length binary data. There's no
way to write a value directly in BQL yet, but there're some ways to use ``blob``
in BQL:

* Emitting a tuple containing a ``blob`` value from a source
* Casting a ``string`` encoded in base64 to ``blob``
* Calling a function returning a ``blob`` value

A ``blob`` value is converted to a base64-encoded string in JSON.

``timestamp``
-------------

The type ``timestamp`` has date and time information in UTC. There's no way to
write a value directly in BQL yet, but there're some ways to use ``blob`` in
BQL:

* Emitting a tuple containing a ``timestamp`` value from a source
* Casting a value of a type that is convertible to ``timestamp``
* Calling a function returning a ``timestamp`` value

A ``timestamp`` value is converted to a string in RFC3339 format with nanosecond
precision in JSON: ``'2006-01-02T15:04:05.999999999Z07:00'``.

``array``
---------

The type ``array`` provides a ordered sequence of values of any type. An
``array`` value is enclosed with brackets (``[`` and ``]``). Each element in an
``array`` is separated by a comma (``,``). A comma after the last element is
allowed. An ``array`` value may have values of different types::

    [1, '2', 3.4]

An ``array`` value can contain another ``array`` or ``map`` as its value::

    [
        [1, '2', 3.4],
        [
            ['4', 5.6, 7],
            [true, false, NULL],
            {'a': 10}
        ],
        {
            'nested_array': [12, 34.5, '67']
        }
    ]

An ``array`` value is converted to an array in JSON.

``map``
-------

The type ``map`` represents an unordered set of key-value pairs. A ``map``
value is enclosed with braces (``{`` and ``}``). Each key-value pair in a ``map``
is separated by a comma (``,``). A comma after the last element isn't allowed.
A key needs to be a ``string`` value and a value can be of any type::

    {
        'a': 1,
        'b': '2',
        'c': 3.4
    }

A ``map`` value can contain another ``map`` or ``array`` as its value::

    {
        'a': {
            'aa': 1,
            'ab': '2',
            'ac': 3.4
        },
        'b': {
            'ba': {'a': 10},
            'bb': ['4', 5.6, 7],
            'bc': [true, false, NULL]
        },
        'c': [12, 34.5, '67']
    }

A ``map`` is converted to an object in JSON.

Conversions
===========

TODO
