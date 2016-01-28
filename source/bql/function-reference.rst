
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

absolute value

Arguments
"""""""""

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

cube root

Arguments
"""""""""

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

smallest integer not less than argument

Arguments
"""""""""

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

radians to degrees

Arguments
"""""""""

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

integer quotient of ``y``/``x``

Arguments
"""""""""

``x``
    2 x ``int`` or 2 x ``float``

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

exponential

Arguments
"""""""""

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

largest integer not greater than argument

Arguments
"""""""""

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

natural logarithm

Arguments
"""""""""

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

Description
"""""""""""

base 10 logarithm

Arguments
"""""""""

``x``
    ``int`` or ``float``

Return Type
"""""""""""

``float``

Examples
""""""""

+----------------+----------+
| Function Call  | Result   |
+================+==========+
| ``log(100)``   | ``2.0``  |
+----------------+----------+




``log``
^^^^^^^

::

    log(b, x)

Description
"""""""""""

logarithm to base b

Arguments
"""""""""

``x``
    2 x ``int`` or 2 x ``float``

Return Type
"""""""""""

``float``

Examples
""""""""

+--------------------+----------+
| Function Call      | Result   |
+====================+==========+
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

remainder of ``y``/``x``

Arguments
"""""""""

``x``
    2 x ``int`` or 2 x ``float``

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

"π" constant

Arguments
"""""""""

``x``
    none

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

``a`` raised to the power of ``b``

Arguments
"""""""""

``x``
    2 x ``int`` or 2 x ``float``

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

Arguments
"""""""""

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

Arguments
"""""""""

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

Arguments
"""""""""

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

Arguments
"""""""""

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

Arguments
"""""""""

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

Arguments
"""""""""

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

Arguments
"""""""""

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

Arguments
"""""""""

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

Arguments
"""""""""

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

Arguments
"""""""""

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

Arguments
"""""""""

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

Arguments
"""""""""

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

Arguments
"""""""""

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

Arguments
"""""""""

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

Arguments
"""""""""

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

Arguments
"""""""""

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

Arguments
"""""""""

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

Arguments
"""""""""

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

Arguments
"""""""""

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

Arguments
"""""""""

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

Arguments
"""""""""

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

Arguments
"""""""""

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

Arguments
"""""""""

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

Arguments
"""""""""

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

Arguments
"""""""""

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

Arguments
"""""""""

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

Arguments
"""""""""

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

Arguments
"""""""""

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

Arguments
"""""""""

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

Arguments
"""""""""

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

Arguments
"""""""""

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

Arguments
"""""""""

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

Arguments
"""""""""

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

Arguments
"""""""""

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

Arguments
"""""""""

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

Arguments
"""""""""

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

Arguments
"""""""""

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

Arguments
"""""""""

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

Arguments
"""""""""

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

Arguments
"""""""""

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

Arguments
"""""""""

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

Arguments
"""""""""

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

Arguments
"""""""""

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

Arguments
"""""""""

``x``
    ``int`` or ``float``

Return Type
"""""""""""

``float`` if the input contains a
``float``, ``int`` otherwise


