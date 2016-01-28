
Function Reference
==================

Scalar Functions Reference
--------------------------


``abs``
^^^^^^^

::

    abs(x)

Description
"""""""""""

``abs`` computes the absolute value of a number.

Parameter Types
"""""""""""""""

``x``
    ``int`` or ``float``

Return Type
"""""""""""

same as input

Examples
""""""""

+----------------+----------+
| Function Call  | Result   |
+================+==========+
| ``abs(-17.4)`` | ``17.4`` |
+----------------+----------+


``cbrt``
^^^^^^^^

::

    cbrt(x)

Description
"""""""""""

``cbrt`` computes the cube root of a number.

Parameter Types
"""""""""""""""

``x``
    ``int`` or ``float``

Return Type
"""""""""""

``float``

Examples
""""""""

+----------------+---------------+
| Function Call  | Result        |
+================+===============+
| ``cbrt(27.0)`` | ``3.0``       |
+----------------+---------------+
| ``cbrt(-3)``   | ``-1.442...`` |
+----------------+---------------+



``ceil``
^^^^^^^^

::

    ceil(x)

Description
"""""""""""

``ceil`` computes the smallest integer not less than its argument.

Parameter Types
"""""""""""""""

``x``
    ``int`` or ``float``

Return Type
"""""""""""

same as input

Examples
""""""""

+----------------+----------+
| Function Call  | Result   |
+================+==========+
| ``ceil(1.3)``  | ``2.0``  |
+----------------+----------+
| ``ceil(-1.7)`` | ``-1.0`` |
+----------------+----------+




``degrees``
^^^^^^^^^^^

::

    degrees(x)

Description
"""""""""""

``degrees`` computes the smallest integer not less than its argument.

Parameter Types
"""""""""""""""

``x``
    ``int`` or ``float``

Return Type
"""""""""""

``float``

Examples
""""""""

+-------------------------+-----------+
| Function Call           | Result    |
+=========================+===========+
| ``degrees(3.14159...)`` | ``180.0`` |
+-------------------------+-----------+




``div``
^^^^^^^

::

    div(y, x)

Description
"""""""""""

``div`` computes the integer quotient ``y``/``x`` of two numbers ``y`` and ``x``.
If ``x`` is ``0.0`` (float) then ``NaN`` will be returned; it it is ``0`` (integer) then a runtime error will occur.

Parameter Types
"""""""""""""""

``y``
    ``int`` or ``float``

``x``
    same as ``y``

Return Type
"""""""""""

same as input

Examples
""""""""

+-------------------+----------+
| Function Call     | Result   |
+===================+==========+
| ``div(9, 4)``     | ``2``    |
+-------------------+----------+
| ``div(9.3, 4.5)`` | ``2.0``  |
+-------------------+----------+



``exp``
^^^^^^^

::

    exp(x)

Description
"""""""""""

``exp`` computes the exponential of a number.

Parameter Types
"""""""""""""""

``x``
    ``int`` or ``float``

Return Type
"""""""""""

``float``

Examples
""""""""

+----------------+---------------+
| Function Call  | Result        |
+================+===============+
| ``exp(1.0)``   | ``2.7182...`` |
+----------------+---------------+



``floor``
^^^^^^^^^

::

    floor(x)

Description
"""""""""""

``floor`` computes the largest integer not greater than its argument.

Parameter Types
"""""""""""""""

``x``
    ``int`` or ``float``

Return Type
"""""""""""

same as input

Examples
""""""""

+-----------------+----------+
| Function Call   | Result   |
+=================+==========+
| ``floor(1.3)``  | ``1.0``  |
+-----------------+----------+
| ``floor(-1.7)`` | ``-2.0`` |
+-----------------+----------+



``ln``
^^^^^^

::

    ln(x)

Description
"""""""""""

``ln`` computes the natural logarithm of a number.
If the parameter is not strictly positive, ``NaN`` is returned.

Parameter Types
"""""""""""""""

``x``
    ``int`` or ``float``

Return Type
"""""""""""

``float``

Examples
""""""""

+----------------+---------------+
| Function Call  | Result        |
+================+===============+
| ``ln(2)``      | ``0.6931...`` |
+----------------+---------------+



``log``
^^^^^^^

::

    log(x)
    log(b, x)

Description
"""""""""""

``log`` computes the logarithm of a number ``x`` to base ``b`` (default: 10).

Parameter Types
"""""""""""""""

``x``
    ``int`` or ``float``

``b`` (optional)
    same as ``x``

Return Type
"""""""""""

``float``

Examples
""""""""

+--------------------+----------+
| Function Call      | Result   |
+====================+==========+
| ``log(100)``       | ``2.0``  |
+--------------------+----------+
| ``log(2.5, 6.25)`` | ``2.0``  |
+--------------------+----------+
| ``log(2, 8)``      | ``3.0``  |
+--------------------+----------+




``mod``
^^^^^^^

::

    mod(y, x)

Description
"""""""""""

``mod`` computes the remainder of integer division ``y``/``x`` of two numbers ``y`` and ``x``.
If ``x`` is ``0.0`` (float) then ``NaN`` will be returned; it it is ``0`` (integer) then a runtime error will occur.

Parameter Types
"""""""""""""""

``y``
    ``int`` or ``float``

``x``
    same as ``y``

Return Type
"""""""""""

same as input

Examples
""""""""

+-------------------+----------+
| Function Call     | Result   |
+===================+==========+
| ``mod(9, 4)``     | ``1``    |
+-------------------+----------+
| ``mod(9.3, 4.5)`` | ``0.3``  |
+-------------------+----------+



``pi``
^^^^^^

::

    pi()

Description
"""""""""""

``pi`` returns the π constant (more or less 3.14).

Return Type
"""""""""""

``float``

Examples
""""""""

+----------------+---------------+
| Function Call  | Result        |
+================+===============+
| ``pi()``       | ``3.1415...`` |
+----------------+---------------+



``power``
^^^^^^^^^

::

    power(a, b)

Description
"""""""""""

``power`` computes ``a`` raised to the power of ``b``.

Parameter Types
"""""""""""""""

``a``
    ``int`` or ``float``

``b``
    same as ``a``

Return Type
"""""""""""

``float``

Examples
""""""""

+---------------------+-----------+
| Function Call       | Result    |
+=====================+===========+
| ``power(9.0, 3.0)`` | ``729.0`` |
+---------------------+-----------+
| ``power(2, -1)``    | ``0.5``   |
+---------------------+-----------+



``radians``
^^^^^^^^^^^

::

    radians(x)

Description
"""""""""""

degrees to radians

Parameter Types
"""""""""""""""

``x``
    ``int`` or ``float``

Return Type
"""""""""""

``float``

Examples
""""""""

+------------------+---------------+
| Function Call    | Result        |
+==================+===============+
| ``radians(180)`` | ``3.1415...`` |
+------------------+---------------+



``round``
^^^^^^^^^

::

    round(x)

Description
"""""""""""

round to nearest integer

Parameter Types
"""""""""""""""

``x``
    ``int`` or ``float``

Return Type
"""""""""""

same as input

Examples
""""""""

+-----------------+----------+
| Function Call   | Result   |
+=================+==========+
| ``round(1.3)``  | ``1.0``  |
+-----------------+----------+
| ``round(-1.7)`` | ``-2.0`` |
+-----------------+----------+



``sqrt``
^^^^^^^^

::

    sqrt(x)

Description
"""""""""""

square root

Parameter Types
"""""""""""""""

``x``
    ``int`` or ``float``

Return Type
"""""""""""

``float``

Examples
""""""""

+----------------+---------------+
| Function Call  | Result        |
+================+===============+
| ``sqrt(2)``    | ``1.4142...`` |
+----------------+---------------+




``trunc``
^^^^^^^^^

::

    trunc(x)

Description
"""""""""""

truncate toward zero

Parameter Types
"""""""""""""""

``x``
    ``int`` or ``float``

Return Type
"""""""""""

same as input

Examples
""""""""

+-----------------+----------+
| Function Call   | Result   |
+=================+==========+
| ``trunc(1.3)``  | ``1.0``  |
+-----------------+----------+
| ``trunc(-1.7)`` | ``-1.0`` |
+-----------------+----------+



``width_bucket``
^^^^^^^^^^^^^^^^

::

    width_bucket(x, l, r, count)

Description
"""""""""""

return the bucket number to which ``x`` would be
assigned in a histogram having ``count`` equal-width
buckets spanning the range ``l`` to ``r``; returns

Parameter Types
"""""""""""""""

``x``
    ``x``, ``l``, ``r``: ``int`` or ``float``

``count``
    ``int``

Return Type
"""""""""""

``int``

Examples
""""""""

+-------------------------------+----------+
| Function Call                 | Result   |
+===============================+==========+
| ``width_bucket(5, 0, 10, 5)`` | ``3``    |
+-------------------------------+----------+




``random``
^^^^^^^^^^

::

    random()

Description
"""""""""""

random value in the range :math:`0.0 <= x < 1.0`

Parameter Types
"""""""""""""""

``x``
    none

Return Type
"""""""""""

``float``



``setseed``
^^^^^^^^^^^

::

    setseed(x)

Description
"""""""""""

set seed (:math:`-1.0 <= x <= 1.0`) for subsequent ``random()`` calls

Parameter Types
"""""""""""""""

``x``
    ``float``

Return Type
"""""""""""

``null``



``acos``
^^^^^^^^

::

    acos(x)

Description
"""""""""""

inverse cosine



``asin``
^^^^^^^^

::

    asin(x)

Description
"""""""""""

inverse sine



``atan``
^^^^^^^^

::

    atan(x)

Description
"""""""""""

inverse tangent



``cos``
^^^^^^^

::

    cos(x)

Description
"""""""""""

cosine



``cot``
^^^^^^^

::

    cot(x)

Description
"""""""""""

cotangent



``sin``
^^^^^^^

::

    sin(x)

Description
"""""""""""

sine




``tan``
^^^^^^^

::

    tan(x)

Description
"""""""""""

tangent



``bit_length``
^^^^^^^^^^^^^^

::

    bit_length(s)

Description
"""""""""""

Number of bits in string

Parameter Types
"""""""""""""""

``x``
    ``string``

Return Type
"""""""""""

``int``

Examples
""""""""

+------------------------+----------+
| Function Call          | Result   |
+========================+==========+
| ``bit_length('über')`` | ``40``   |
+------------------------+----------+



``btrim``
^^^^^^^^^

::

    btrim(s)

Description
"""""""""""

Remove whitespace from the start and end of ``s``

Parameter Types
"""""""""""""""

``x``
    ``string``

Return Type
"""""""""""

``string``

Examples
""""""""

+-----------------------+------------+
| Function Call         | Result     |
+=======================+============+
| ``btrim('  trim  ')`` | ``'trim'`` |
+-----------------------+------------+



``btrim``
^^^^^^^^^

::

    btrim(s, chars)

Description
"""""""""""

Remove the longest string consisting only of characters in ``chars``
from the start and end of ``s``

Parameter Types
"""""""""""""""

``x``
    2 x ``string``

Return Type
"""""""""""

``string``

Examples
""""""""

+-------------------------------+------------+
| Function Call                 | Result     |
+===============================+============+
| ``btrim('xyxtrimyyx', 'xy')`` | ``'trim'`` |
+-------------------------------+------------+



``char_length``
^^^^^^^^^^^^^^^

::

    char_length(s)

Description
"""""""""""

Number of characters in ``s``

Parameter Types
"""""""""""""""

``x``
    ``string``

Return Type
"""""""""""

``int``

Examples
""""""""

+-------------------------+----------+
| Function Call           | Result   |
+=========================+==========+
| ``char_length('über')`` | ``4``    |
+-------------------------+----------+



``concat``
^^^^^^^^^^

::

    concat(s [, ...])

Description
"""""""""""

Concatenate the text representations of all the arguments.
NULL arguments are ignored.

Parameter Types
"""""""""""""""

``x``
    *n* x ``string``

Return Type
"""""""""""

``string``

Examples
""""""""

+-------------------------------+-------------+
| Function Call                 | Result      |
+===============================+=============+
| ``concat('abc', NULL, '22')`` | ``'abc22'`` |
+-------------------------------+-------------+



``format``
^^^^^^^^^^

::

    format(s, [x, ...])

Description
"""""""""""

Format arguments according to a format string.
This function is similar to the Go function ``fmt.Sprintf``.

Parameter Types
"""""""""""""""

``x``
    ``string``, *n* x any

Return Type
"""""""""""

``string``

Examples
""""""""

+--------------------------------+--------------+
| Function Call                  | Result       |
+================================+==============+
| ``format('%s-%d', 'abc', 22)`` | ``'abc-22'`` |
+--------------------------------+--------------+



``lower``
^^^^^^^^^

::

    lower(s)

Description
"""""""""""

Convert ``s`` to lower case

Parameter Types
"""""""""""""""

``x``
    ``string``

Return Type
"""""""""""

``string``

Examples
""""""""

+-------------------+------------+
| Function Call     | Result     |
+===================+============+
| ``lower('ÜBer')`` | ``'über'`` |
+-------------------+------------+




``ltrim``
^^^^^^^^^

::

    ltrim(s)

Description
"""""""""""

Remove whitespace from the start of ``s``

Parameter Types
"""""""""""""""

``x``
    ``string``

Return Type
"""""""""""

``string``

Examples
""""""""

+-----------------------+--------------+
| Function Call         | Result       |
+=======================+==============+
| ``ltrim('  trim  ')`` | ``'trim  '`` |
+-----------------------+--------------+



``ltrim``
^^^^^^^^^

::

    ltrim(s, chars)

Description
"""""""""""

Remove the longest string consisting only of characters in ``chars``
from the start of ``s``

Parameter Types
"""""""""""""""

``x``
    2 x ``string``

Return Type
"""""""""""

``string``

Examples
""""""""

+-------------------------------+-------------+
| Function Call                 | Result      |
+===============================+=============+
| ``ltrim('xyxtrimyyx', 'xy')`` | ``trimyyx`` |
+-------------------------------+-------------+



``md5``
^^^^^^^

::

    md5(s)

Description
"""""""""""

Calculates the MD5 hash of ``s``, returning the result in hexadecimal

Parameter Types
"""""""""""""""

``x``
    ``string``

Return Type
"""""""""""

``string``

Examples
""""""""

+----------------+----------------------------------------+
| Function Call  | Result                                 |
+================+========================================+
| ``md5('abc')`` | ``'900150983cd24fb0d6963f7d28e17f72'`` |
+----------------+----------------------------------------+



``octet_length``
^^^^^^^^^^^^^^^^

::

    octet_length(s)

Description
"""""""""""

Number of bytes in ``s``

Parameter Types
"""""""""""""""

``x``
    ``string``

Return Type
"""""""""""

``int``

Examples
""""""""

+--------------------------+----------+
| Function Call            | Result   |
+==========================+==========+
| ``octet_length('über')`` | ``5``    |
+--------------------------+----------+



``overlay``
^^^^^^^^^^^

::

    overlay(s, r, from)

Description
"""""""""""

Replace substring

Parameter Types
"""""""""""""""

``x``
    2 x ``string``, ``int``

Return Type
"""""""""""

``string``

Examples
""""""""

+----------------------------------+---------------+
| Function Call                    | Result        |
+==================================+===============+
| ``overlay('Txxxxas', 'hom', 2)`` | ``'Thomxas'`` |
+----------------------------------+---------------+




``overlay``
^^^^^^^^^^^

::

    overlay(s, r, from, for)

Description
"""""""""""

Replace substring

Parameter Types
"""""""""""""""

``x``
    2 x ``string``, 2 x ``int``

Return Type
"""""""""""

``string``

Examples
""""""""

+-------------------------------------+--------------+
| Function Call                       | Result       |
+=====================================+==============+
| ``overlay('Txxxxas', 'hom', 2, 4)`` | ``'Thomas'`` |
+-------------------------------------+--------------+



``rtrim``
^^^^^^^^^

::

    rtrim(s)

Description
"""""""""""

Remove whitespace from the end of ``s``

Parameter Types
"""""""""""""""

``x``
    ``string``

Return Type
"""""""""""

``string``

Examples
""""""""

+-----------------------+--------------+
| Function Call         | Result       |
+=======================+==============+
| ``rtrim('  trim  ')`` | ``'  trim'`` |
+-----------------------+--------------+



``rtrim``
^^^^^^^^^

::

    rtrim(s, chars)

Description
"""""""""""

Remove the longest string consisting only of characters in ``chars``
from the end of ``s``

Parameter Types
"""""""""""""""

``x``
    2 x ``string``

Return Type
"""""""""""

``string``

Examples
""""""""

+-------------------------------+-------------+
| Function Call                 | Result      |
+===============================+=============+
| ``rtrim('xyxtrimyyx', 'xy')`` | ``xyxtrim`` |
+-------------------------------+-------------+



``sha1``
^^^^^^^^

::

    sha1(s)

Description
"""""""""""

Calculates the SHA1 hash of ``s``, returning the result in hexadecimal

Parameter Types
"""""""""""""""

``x``
    ``string``

Return Type
"""""""""""

``string``

Examples
""""""""

+-----------------+------------------------------------------------+
| Function Call   | Result                                         |
+=================+================================================+
| ``sha1('abc')`` | ``'a9993e364706816aba3e25717850c26c9cd0d89d'`` |
+-----------------+------------------------------------------------+



``sha256``
^^^^^^^^^^

::

    sha256(s)

Description
"""""""""""

Calculates the SHA256 hash of ``s``, returning the result in hexadecimal

Parameter Types
"""""""""""""""

``x``
    ``string``

Return Type
"""""""""""

``string``

Examples
""""""""

+-------------------+------------------------------------------------------------------------+
| Function Call     | Result                                                                 |
+===================+========================================================================+
| ``sha256('abc')`` | ``'ba7816bf8f01cfea414140de5dae2223b00361a396177a9cb410ff61f20015ad'`` |
+-------------------+------------------------------------------------------------------------+




``strpos``
^^^^^^^^^^

::

    strpos(s, t)

Description
"""""""""""

Location of specified substring ``t`` in ``s``

Parameter Types
"""""""""""""""

``x``
    2 x ``string``

Return Type
"""""""""""

``int``

Examples
""""""""

+--------------------------+----------+
| Function Call            | Result   |
+==========================+==========+
| ``strpos('high', 'ig')`` | ``2``    |
+--------------------------+----------+




``substring``
^^^^^^^^^^^^^

::

    substring(s, r)

Description
"""""""""""

Extract substring matching regular expression ``r`` from ``s``.
See Go ``regexp`` package for details of matching.

Parameter Types
"""""""""""""""

``x``
    2 x ``string``

Return Type
"""""""""""

``string``

Examples
""""""""

+---------------------------------+-----------+
| Function Call                   | Result    |
+=================================+===========+
| ``substring('Thomas', '...$')`` | ``'mas'`` |
+---------------------------------+-----------+



``substring``
^^^^^^^^^^^^^

::

    substring(s, from)

Description
"""""""""""

Extract substring

Parameter Types
"""""""""""""""

``x``
    ``string``, ``int``

Return Type
"""""""""""

``string``

Examples
""""""""

+----------------------------+-------------+
| Function Call              | Result      |
+============================+=============+
| ``substring('Thomas', 2)`` | ``'homas'`` |
+----------------------------+-------------+



``substring``
^^^^^^^^^^^^^

::

    substring(s, from, for)

Description
"""""""""""

Extract substring

Parameter Types
"""""""""""""""

``x``
    ``string``, 2 x ``int``

Return Type
"""""""""""

``string``

Examples
""""""""

+-------------------------------+-----------+
| Function Call                 | Result    |
+===============================+===========+
| ``substring('Thomas', 2, 3)`` | ``'hom'`` |
+-------------------------------+-----------+



``upper``
^^^^^^^^^

::

    upper(s)

Description
"""""""""""

Convert ``s`` to upper case

Parameter Types
"""""""""""""""

``x``
    ``string``

Return Type
"""""""""""

``string``

Examples
""""""""

+-------------------+------------+
| Function Call     | Result     |
+===================+============+
| ``upper('ÜBer')`` | ``'ÜBER'`` |
+-------------------+------------+



``distance_us``
^^^^^^^^^^^^^^^

::

    distance_us(u, v)

Description
"""""""""""

Signed temporal distance from ``u`` to ``v`` in microseconds

Parameter Types
"""""""""""""""

``x``
    2 x ``timestamp``

Return Type
"""""""""""

``int``



``clock_timestamp``
^^^^^^^^^^^^^^^^^^^

::

    ``clock_timestamp()``

Description
"""""""""""

Current date and time (changes during statement execution)

Parameter Types
"""""""""""""""

``x``
    none

Return Type
"""""""""""

``timestamp``




``coalesce``
^^^^^^^^^^^^

::

    coalesce([x, ...])

Description
"""""""""""

Returns the first non-null input parameter
or NULL if there is no such parameter

Parameter Types
"""""""""""""""

``x``
    *n* x any

Return Type
"""""""""""

same as input

Examples
""""""""

+-------------------------------+----------+
| Function Call                 | Result   |
+===============================+==========+
| ``coalesce(NULL, 17, 'foo')`` | ``17``   |
+-------------------------------+----------+




Aggregate Functions Reference
-----------------------------


``array_agg``
^^^^^^^^^^^^^

::

    array_agg(x)

Description
"""""""""""

input values, including nulls, concatenated into an array

Parameter Types
"""""""""""""""

``x``
    any

Return Type
"""""""""""

``array``



``avg``
^^^^^^^

::

    avg(x)

Description
"""""""""""

the average (arithmetic mean) of all input values

Parameter Types
"""""""""""""""

``x``
    ``int`` or ``float``

Return Type
"""""""""""

``float``



``bool_and``
^^^^^^^^^^^^

::

    bool_and(x)

Description
"""""""""""

true if all input values are true, otherwise false

Parameter Types
"""""""""""""""

``x``
    ``bool``

Return Type
"""""""""""

``bool``



``bool_or``
^^^^^^^^^^^

::

    bool_or(x)

Description
"""""""""""

true if at least one input value is true, otherwise false

Parameter Types
"""""""""""""""

``x``
    ``bool``

Return Type
"""""""""""

``bool``




``count``
^^^^^^^^^

::

    count(x)

Description
"""""""""""

number of input rows for which ``x`` is not null

Parameter Types
"""""""""""""""

``x``
    any

Return Type
"""""""""""

``int``



``count``
^^^^^^^^^

::

    count(*)

Description
"""""""""""

number of input rows

Parameter Types
"""""""""""""""

``x``
    none

Return Type
"""""""""""

``int``



``json_object_agg``
^^^^^^^^^^^^^^^^^^^

::

    json_object_agg(k, v)

Description
"""""""""""

aggregates name/value pairs as a map

Parameter Types
"""""""""""""""

``x``
    ``string``, any

Return Type
"""""""""""

``map``




``max``
^^^^^^^

::

    max(x)

Description
"""""""""""

maximum value of ``x`` across all input values

Parameter Types
"""""""""""""""

``x``
    ``int`` or ``float``

Return Type
"""""""""""

same as largest input value




``median``
^^^^^^^^^^

::

    median(x)

Description
"""""""""""

the median of all input values

Parameter Types
"""""""""""""""

``x``
    ``int`` or ``float``

Return Type
"""""""""""

``float``



``min``
^^^^^^^

::

    min(x)

Description
"""""""""""

minimum value of ``x`` across all input values

Parameter Types
"""""""""""""""

``x``
    ``int`` or ``float``

Return Type
"""""""""""

same as smallest input value



``string_agg``
^^^^^^^^^^^^^^

::

    string_agg(x, sep)

Description
"""""""""""

input values concatenated into a string, separated by ``sep``

Parameter Types
"""""""""""""""

``x``
    ``string``, ``string`` (scalar)

Return Type
"""""""""""

``string``




``sum``
^^^^^^^

::

    sum(x)

Description
"""""""""""

sum of ``x`` across all input values

Parameter Types
"""""""""""""""

``x``
    ``int`` or ``float``

Return Type
"""""""""""

``float`` if the input contains a
``float``, ``int`` otherwise


