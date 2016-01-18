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

The type ``timestamp`` has date and time information in UTC. ``timestamp`` only
guarantees precision in microseconds. There's no way to write a value directly
in BQL yet, but there're some ways to use ``blob`` in BQL:

* Emitting a tuple containing a ``timestamp`` value from a source
* Casting a value of a type that is convertible to ``timestamp``
* Calling a function returning a ``timestamp`` value

A ``timestamp`` value is converted to a string in RFC3339 format with nanosecond
precision in JSON: ``'2006-01-02T15:04:05.999999999Z07:00'``. Although the
format can express nanoseconds, ``timestamp`` in BQL only guarantees microsecond
precision as described above.

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

Because this is a ``map`` in BQL, not a JSON object, keys need to be enclosed
with single quotes rather than double quotes.

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

BQL provides ``CAST(value AS type)`` operator, or ``value::type`` as a syntactic
suger, that converts the given value to a corresponding value in the given type,
if those types are convertible. For example, ``CAST(1 AS string)``, or
``1::string``, converts an ``int`` value ``1`` to a ``string`` value and
results in ``'1'``. Converting to the same type as the value's type is valid.
For instance, ``'str'::string`` doesn't do anything and results in ``'str'``.

The following types are valid for the target type of ``CAST`` operator:

* ``bool``
* ``int``
* ``float``
* ``string``
* ``blob``
* ``timestamp``

Specifying ``null``, ``array``, or ``map`` as the target type results in an
error.

This section describes how type conversions work in BQL.

.. note::

    Converting a ``NULL`` value into any type results in ``NULL`` and it isn't
    explicitly described in the subsections.

To ``bool``
-----------

Following types can be converted to ``bool``:

* ``int``
* ``float``
* ``string``
* ``blob``
* ``timestamp``
* ``array``
* ``map``

From ``int``
^^^^^^^^^^^^

``0`` is converted to ``false``. Other values are converted to ``true``.

From ``float``
^^^^^^^^^^^^^^

``0.0``, ``-0.0``, and NaN are converted to ``false``. Other values including
infinity result in ``true``.

From ``string``
^^^^^^^^^^^^^^^

An empty ``string`` value (i.e. ``''``) is converted to ``false``. Other values
are evaluated as ``true``.

From ``blob``
^^^^^^^^^^^^^

An empty ``blob`` value is converted to ``false``. Other values are converted
to ``true``.

From ``timestamp``
^^^^^^^^^^^^^^^^^^

January 1, year 1, 00:00:00 UTC is converted to ``false``. Other values are
converted to ``true``.

From ``array``
^^^^^^^^^^^^^^

An empty ``array`` is converted to ``false``. Other values result in ``true``.

From ``map``
^^^^^^^^^^^^

An empty ``map`` is converted to ``false``. Other values result in ``true``.

To ``int``
----------

Following types can be converted to ``int``:

* ``bool``
* ``float``
* ``string``
* ``timestamp``

From ``bool``
^^^^^^^^^^^^^

``true::int`` results in 1 and ``false::int`` results in 0.

From ``float``
^^^^^^^^^^^^^^

Converting a ``float`` value into a ``int`` value results in the greatest
``int`` value less than or equal to the ``float`` value::

    1.0::int  -- => 1
    1.4::int  -- => 1
    1.5::int  -- => 1
    2.01::int -- => 2

The conversion results in an error when the ``float`` value is out of the valid
range of ``int`` values.

From ``string``
^^^^^^^^^^^^^^^

When converting a ``string`` value into an ``int`` value, ``CAST`` operator
first tries to parse it as an integer to guarantee better precision. If the
parsing fails, it tries to parse the ``string`` value as a ``float`` value and
then converts the result to an ``int`` value.

::

    '1'::int   -- => 1
    '2.5'::int -- => 2

The conversion results in an error when the ``string`` value contains the
number that is out of the valid range of ``int`` values, or the value isn't a
number. For example, ``'1a'::string`` results in an error even though the value
starts with a number.

From ``timestamp``
^^^^^^^^^^^^^^^^^^

A ``timestamp`` value is converted to an ``int`` value as the number of
microseconds elapsed since January 1, 1970 UTC::

    ('1970-01-01T00:00:00Z'::timestamp)::int        -- => 0
    ('1970-01-01T00:00:00.123456Z'::timestamp)::int -- => 123456
    ('1970-01-02T00:00:00Z'::timestamp)::int        -- => 86400000000
    ('2016-01-18T09:22:40.123456Z'::timestamp)::int -- => 1453108960123456

The maximum ``timestamp`` that can be converted to ``int`` is
294247-01-10T04:00:54.775807Z. The minimum is -290308-12-21T19:59:05.224192Z.

To ``float``
------------

Following types can be converted to ``float``:

* ``bool``
* ``int``
* ``string``
* ``timestamp``

From ``bool``
^^^^^^^^^^^^^

``true::float`` results in 1.0 and ``false::float`` results in 0.0.

From ``int``
^^^^^^^^^^^^

``int`` values are converted to the nearest ``float`` values::

    1::float -- => 1.0
    (9000000000000012345::float)::int)::string -- => '9000000000000012288'

From ``string``
^^^^^^^^^^^^^^^

A ``string`` value is parsed and converted to the nearest ``float`` value::

    '1.1'::float   -- => 1.1
    '1e-1'::float  -- => 0.1
    '-1e+1'::float -- => -10.0

From ``timestamp``
^^^^^^^^^^^^^^^^^^

A ``timestamp`` value is converted to a ``float`` value as the number of
microseconds elapsed since January 1, 1970 UTC. The integral part of the result
contains seconds and the decimal part contains microseconds::

    ('1970-01-01T00:00:00Z'::timestamp)::float        -- => 0.0
    ('1970-01-01T00:00:00.000001Z'::timestamp)::float -- => 0.000001
    ('1970-01-02T00:00:00.000001Z'::timestamp)::float -- => 86400.000001

To ``string``
-------------

Following types can be converted to ``string``:

* ``bool``
* ``int``
* ``float``
* ``blob``
* ``timestamp``
* ``array``
* ``map``

From ``bool``
^^^^^^^^^^^^^

``true::string`` results in ``'true'``, ``false::string`` results in ``'false'``.

From ``int``
^^^^^^^^^^^^

A ``int`` value is formatted as a signed decimal integer::

    1::string     -- => '1'
    (-24)::string -- => '-24'

From ``float``
^^^^^^^^^^^^^^

A ``float`` value is formatted as a signed decimal floating point. Scientific
notation is used when necessary::

    1.2::string           -- => '1.2'
    10000000000.0::string -- => '1e+10'

From ``blob``
^^^^^^^^^^^^^

A ``blob`` value is converted to a ``string`` value encoded in base64.

From ``timestamp``
^^^^^^^^^^^^^^^^^^

A ``timestamp`` value is formatted in RFC3339 format with nanosecond precision:
'2006-01-02T15:04:05.999999999Z07:00'.

From ``array``
^^^^^^^^^^^^^^

An ``array`` value is formatted as a JSON array::

    [1, '2', 3.4]::string -- => '[1,"2",3.4]'

From ``map``
^^^^^^^^^^^^

A ``map`` value is formatted as a JSON object::

    {'a': 1, 'b': '2', 'c': 3.4}::string -- => '{"a":1,"b":"2","c":3.4}'

To ``timestamp``
----------------

Following types can be converted to ``timestamp``:

* ``int``
* ``float``
* ``string``

From ``int``
^^^^^^^^^^^^

An ``int`` value to be converted to a ``timestamp`` value is assumed to have
the number of microseconds elapsed since January 1, 1970 UTC::

    0::timestamp                -- => 1970-01-01T00:00:00Z
    1::timestamp                -- => 1970-01-01T00:00:00.000001Z
    1453108960123456::timestamp -- => 2016-01-18T09:22:40.123456Z

From ``float``
^^^^^^^^^^^^^^

An ``float`` value to be converted to a ``timestamp`` value is assumed to have
the number of microseconds elapsed since January 1, 1970 UTC. Its integral
part should have seconds and decimal part should have microseconds::

    0.0::timestamp -- => 1970-01-01T00:00:00Z
    0.000001::timestamp -- => 1970-01-01T00:00:00.000001Z
    86400.000001::timestamp -- => 1970-01-02T00:00:00.000001Z

From ``string``
^^^^^^^^^^^^^^^

A ``string`` value is parsed in RFC3339 format, or RFC3339 with nanosecond
precision format::

    '1970-01-01T00:00:00Z'::timestamp        -- => 1970-01-01T00:00:00Z
    '1970-01-01T00:00:00.000001Z'::timestamp -- => 1970-01-01T00:00:00.000001Z
    '1970-01-02T00:00:00.000001Z'::timestamp -- => 1970-01-02T00:00:00.000001Z

Converting ill-formed ``string`` values to ``timestamp`` results in an error.
