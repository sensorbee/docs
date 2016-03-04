.. _ref_func:

******************
Function Reference
******************

Scalar Functions Reference
==========================


.. _ref_func_abs:

``abs``
-------

::

    abs(x)

Description
^^^^^^^^^^^

``abs`` computes the absolute value of a number.

Parameter Types
^^^^^^^^^^^^^^^

``x``
    ``int`` or ``float``

Return Type
^^^^^^^^^^^

same as input

Examples
^^^^^^^^

+----------------+----------+
| Function Call  | Result   |
+================+==========+
| ``abs(-17.4)`` | ``17.4`` |
+----------------+----------+


.. _ref_func_cbrt:

``cbrt``
--------

::

    cbrt(x)

Description
^^^^^^^^^^^

``cbrt`` computes the cube root of a number.

Parameter Types
^^^^^^^^^^^^^^^

``x``
    ``int`` or ``float``

Return Type
^^^^^^^^^^^

``float``

Examples
^^^^^^^^

+----------------+-------------------------+
| Function Call  | Result                  |
+================+=========================+
| ``cbrt(27.0)`` | ``3.0``                 |
+----------------+-------------------------+
| ``cbrt(-3)``   | ``-1.4422495703074083`` |
+----------------+-------------------------+



.. _ref_func_ceil:

``ceil``
--------

::

    ceil(x)

Description
^^^^^^^^^^^

``ceil`` computes the smallest integer not less than its argument.

Parameter Types
^^^^^^^^^^^^^^^

``x``
    ``int`` or ``float``

Return Type
^^^^^^^^^^^

same as input

The return type is ``float`` for ``float`` input in order to avoid problems with input values that are too large for the ``int`` data type.

Examples
^^^^^^^^

+----------------+----------+
| Function Call  | Result   |
+================+==========+
| ``ceil(1.3)``  | ``2.0``  |
+----------------+----------+
| ``ceil(-1.7)`` | ``-1.0`` |
+----------------+----------+




.. _ref_func_degrees:

``degrees``
-----------

::

    degrees(x)

Description
^^^^^^^^^^^

``degrees`` converts radians to degrees.

Parameter Types
^^^^^^^^^^^^^^^

``x``
    ``int`` or ``float``

Return Type
^^^^^^^^^^^

``float``

Examples
^^^^^^^^

+--------------------------------+-----------+
| Function Call                  | Result    |
+================================+===========+
| ``degrees(3.141592653589793)`` | ``180.0`` |
+--------------------------------+-----------+




.. _ref_func_div:

``div``
-------

::

    div(y, x)

Description
^^^^^^^^^^^

``div`` computes the integer quotient ``y``/``x`` of two numbers ``y`` and ``x``.
If ``x`` is ``0.0`` (float) then ``NaN`` will be returned; it it is ``0`` (integer) then a runtime error will occur.

Parameter Types
^^^^^^^^^^^^^^^

``y``
    ``int`` or ``float``

``x``
    same as ``y``

Return Type
^^^^^^^^^^^

same as input

Examples
^^^^^^^^

+-------------------+----------+
| Function Call     | Result   |
+===================+==========+
| ``div(9, 4)``     | ``2``    |
+-------------------+----------+
| ``div(9.3, 4.5)`` | ``2.0``  |
+-------------------+----------+



.. _ref_func_exp:

``exp``
-------

::

    exp(x)

Description
^^^^^^^^^^^

``exp`` computes the exponential of a number.

Parameter Types
^^^^^^^^^^^^^^^

``x``
    ``int`` or ``float``

Return Type
^^^^^^^^^^^

``float``

Examples
^^^^^^^^

+----------------+-----------------------+
| Function Call  | Result                |
+================+=======================+
| ``exp(1.0)``   | ``2.718281828459045`` |
+----------------+-----------------------+



.. _ref_func_floor:

``floor``
---------

::

    floor(x)

Description
^^^^^^^^^^^

``floor`` computes the largest integer not greater than its argument.

Parameter Types
^^^^^^^^^^^^^^^

``x``
    ``int`` or ``float``

Return Type
^^^^^^^^^^^

same as input

The return type is ``float`` for ``float`` input in order to avoid problems with input values that are too large for the ``int`` data type.

Examples
^^^^^^^^

+-----------------+----------+
| Function Call   | Result   |
+=================+==========+
| ``floor(1.3)``  | ``1.0``  |
+-----------------+----------+
| ``floor(-1.7)`` | ``-2.0`` |
+-----------------+----------+



.. _ref_func_ln:

``ln``
------

::

    ln(x)

Description
^^^^^^^^^^^

``ln`` computes the natural logarithm of a number.
If the parameter is not strictly positive, ``NaN`` is returned.

Parameter Types
^^^^^^^^^^^^^^^

``x``
    ``int`` or ``float``

Return Type
^^^^^^^^^^^

``float``

Examples
^^^^^^^^

+----------------+------------------------+
| Function Call  | Result                 |
+================+========================+
| ``ln(2)``      | ``0.6931471805599453`` |
+----------------+------------------------+



.. _ref_func_log:

``log``
-------

::

    log(x)
    log(b, x)

Description
^^^^^^^^^^^

``log`` computes the logarithm of a number ``x`` to base ``b`` (default: 10).

Parameter Types
^^^^^^^^^^^^^^^

``x``
    ``int`` or ``float``

``b`` (optional)
    same as ``x``

Return Type
^^^^^^^^^^^

``float``

Examples
^^^^^^^^

+--------------------+----------+
| Function Call      | Result   |
+====================+==========+
| ``log(100)``       | ``2.0``  |
+--------------------+----------+
| ``log(2.5, 6.25)`` | ``2.0``  |
+--------------------+----------+
| ``log(2, 8)``      | ``3.0``  |
+--------------------+----------+




.. _ref_func_mod:

``mod``
-------

::

    mod(y, x)

Description
^^^^^^^^^^^

``mod`` computes the remainder of integer division ``y``/``x`` of two numbers ``y`` and ``x``.
If ``x`` is ``0.0`` (float) then ``NaN`` will be returned; it it is ``0`` (integer) then a runtime error will occur.

Parameter Types
^^^^^^^^^^^^^^^

``y``
    ``int`` or ``float``

``x``
    same as ``y``

Return Type
^^^^^^^^^^^

same as input

Examples
^^^^^^^^

+-------------------+----------+
| Function Call     | Result   |
+===================+==========+
| ``mod(9, 4)``     | ``1``    |
+-------------------+----------+
| ``mod(9.3, 4.5)`` | ``0.3``  |
+-------------------+----------+



.. _ref_func_pi:

``pi``
------

::

    pi()

Description
^^^^^^^^^^^

``pi`` returns the π constant (more or less 3.14).

Return Type
^^^^^^^^^^^

``float``

Examples
^^^^^^^^

+----------------+-----------------------+
| Function Call  | Result                |
+================+=======================+
| ``pi()``       | ``3.141592653589793`` |
+----------------+-----------------------+



.. _ref_func_power:

``power``
---------

::

    power(a, b)

Description
^^^^^^^^^^^

``power`` computes ``a`` raised to the power of ``b``.

Parameter Types
^^^^^^^^^^^^^^^

``a``
    ``int`` or ``float``

``b``
    same as ``a``

Return Type
^^^^^^^^^^^

``float``

The return type is ``float`` even for integer input in order to have a uniform behavior for cases such as ``power(2, -2)``.

Examples
^^^^^^^^

+---------------------+-----------+
| Function Call       | Result    |
+=====================+===========+
| ``power(9.0, 3.0)`` | ``729.0`` |
+---------------------+-----------+
| ``power(2, -1)``    | ``0.5``   |
+---------------------+-----------+



.. _ref_func_radians:

``radians``
-----------

::

    radians(x)

Description
^^^^^^^^^^^

``radians`` converts degrees to radians.

Parameter Types
^^^^^^^^^^^^^^^

``x``
    ``int`` or ``float``

Return Type
^^^^^^^^^^^

``float``

Examples
^^^^^^^^

+------------------+-----------------------+
| Function Call    | Result                |
+==================+=======================+
| ``radians(180)`` | ``3.141592653589793`` |
+------------------+-----------------------+



.. _ref_func_round:

``round``
---------

::

    round(x)

Description
^^^^^^^^^^^

``round`` computes the nearest integer of a number.

Parameter Types
^^^^^^^^^^^^^^^

``x``
    ``int`` or ``float``

Return Type
^^^^^^^^^^^

same as input

The return type is ``float`` for ``float`` input in order to avoid problems with input values that are too large for the ``int`` data type.

Examples
^^^^^^^^

+-----------------+----------+
| Function Call   | Result   |
+=================+==========+
| ``round(1.3)``  | ``1.0``  |
+-----------------+----------+
| ``round(0.5)``  | ``1.0``  |
+-----------------+----------+
| ``round(-1.7)`` | ``-2.0`` |
+-----------------+----------+



.. _ref_func_sign:

``sign``
--------

::

    sign(x)

Description
^^^^^^^^^^^

``sign`` returns the sign of a number: 1 for positive numbers, -1 for negative numbers and 0 for zero.

Parameter Types
^^^^^^^^^^^^^^^

``x``
    ``int`` or ``float``

Return Type
^^^^^^^^^^^

``int``

Examples
^^^^^^^^

+----------------+------------------------+
| Function Call  | Result                 |
+================+========================+
| ``sign(2)``    | ``1``                  |
+----------------+------------------------+



.. _ref_func_sqrt:

``sqrt``
--------

::

    sqrt(x)

Description
^^^^^^^^^^^

``sqrt`` computes the square root of a number.
If the parameter is negative, ``NaN`` is returned.

Parameter Types
^^^^^^^^^^^^^^^

``x``
    ``int`` or ``float``

Return Type
^^^^^^^^^^^

``float``

Examples
^^^^^^^^

+----------------+------------------------+
| Function Call  | Result                 |
+================+========================+
| ``sqrt(2)``    | ``1.4142135623730951`` |
+----------------+------------------------+




.. _ref_func_trunc:

``trunc``
---------

::

    trunc(x)

Description
^^^^^^^^^^^

``trunc`` computes the truncated integer (towards zero) of a number.

Parameter Types
^^^^^^^^^^^^^^^

``x``
    ``int`` or ``float``

Return Type
^^^^^^^^^^^

same as input

The return type is ``float`` for ``float`` input in order to avoid problems with input values that are too large for the ``int`` data type.

Examples
^^^^^^^^

+-----------------+----------+
| Function Call   | Result   |
+=================+==========+
| ``trunc(1.3)``  | ``1.0``  |
+-----------------+----------+
| ``trunc(-1.7)`` | ``-1.0`` |
+-----------------+----------+



.. _ref_func_width_bucket:

``width_bucket``
----------------

::

    width_bucket(x, left, right, count)

Description
^^^^^^^^^^^

``widthBucketFunc`` computes the bucket to which ``x`` would be assigned in an equidepth histogram with ``count`` buckets in the range :math:`[\text{left},\text{right}[`.
Points on a bucket border belong to the right bucket.
Points outside of the :math:`[\text{left},\text{right}[` range have bucket number :math:`0` and :math:`\text{count}+1`, respectively.

Parameter Types
^^^^^^^^^^^^^^^

``x``
    ``int`` or ``float``

``left``
    ``int`` or ``float``

``right``
    ``int`` or ``float``

``count``
    ``int``

Return Type
^^^^^^^^^^^

``int``

Examples
^^^^^^^^

+-------------------------------+----------+
| Function Call                 | Result   |
+===============================+==========+
| ``width_bucket(5, 0, 10, 5)`` | ``3``    |
+-------------------------------+----------+




.. _ref_func_random:

``random``
----------

::

    random()

Description
^^^^^^^^^^^

``random`` returns a pseudo-random number in the range :math:`0.0 <= x < 1.0`.

This function is not safe for use in cryptographic applications.
See the `Go math/rand package <https://golang.org/pkg/math/rand/>`_ for details.

Return Type
^^^^^^^^^^^

``float``

Examples
^^^^^^^^

+---------------+--------------------------+
| Function Call | Result                   |
+===============+==========================+
| ``random()``  | ``0.6046602879796196``   |
+---------------+--------------------------+



.. _ref_func_setseed:

``setseed``
-----------

::

    setseed(x)

Description
^^^^^^^^^^^

``setseed`` initializes the seed for subsequent ``random()`` calls.
The parameter must be in the range :math:`-1.0 <= x <= 1.0`.

This function is not safe for use in cryptographic applications.
See the `Go math/rand package <https://golang.org/pkg/math/rand/>`_ for details.

Parameter Types
^^^^^^^^^^^^^^^

``x``
    ``float``



.. _ref_func_acos:

``acos``
--------

::

    acos(x)

Description
^^^^^^^^^^^

``acos`` computes the inverse cosine of a number.



.. _ref_func_asin:

``asin``
--------

::

    asin(x)

Description
^^^^^^^^^^^

``asin`` computes the inverse sine of a number.



.. _ref_func_atan:

``atan``
--------

::

    atan(x)

Description
^^^^^^^^^^^

``atan`` computes the inverse tangent of a number.



.. _ref_func_cos:

``cos``
-------

::

    cos(x)

Description
^^^^^^^^^^^

``cos`` computes the cosine of a number.



.. _ref_func_cot:

``cot``
-------

::

    cot(x)

Description
^^^^^^^^^^^

``cot`` computes the cotangent of a number.



.. _ref_func_sin:

``sin``
-------

::

    sin(x)

Description
^^^^^^^^^^^

``sin`` computes the sine of a number.




.. _ref_func_tan:

``tan``
-------

::

    tan(x)

Description
^^^^^^^^^^^

``tan`` computes the tangent of a number.



.. _ref_func_bit_length:

``bit_length``
--------------

::

    bit_length(s)

Description
^^^^^^^^^^^

``bit_length`` computes the number of bits in a string.

Parameter Types
^^^^^^^^^^^^^^^

``s``
    ``string``

Return Type
^^^^^^^^^^^

``int``

Examples
^^^^^^^^

+------------------------+----------+
| Function Call          | Result   |
+========================+==========+
| ``bit_length("über")`` | ``40``   |
+------------------------+----------+



.. _ref_func_btrim:

``btrim``
---------

::

    btrim(s)
    btrim(s, chars)

Description
^^^^^^^^^^^

``btrim`` removes the longest string consisting only of characters in ``chars`` (default: whitespace) from the start and end of ``s``.

Parameter Types
^^^^^^^^^^^^^^^

``s``
    ``string``

``chars`` (optional)
    ``string``

Return Type
^^^^^^^^^^^

``string``

Examples
^^^^^^^^

+-------------------------------+------------+
| Function Call                 | Result     |
+===============================+============+
| ``btrim("  trim  ")``         | ``"trim"`` |
+-------------------------------+------------+
| ``btrim("xyxtrimyyx", "xy")`` | ``"trim"`` |
+-------------------------------+------------+



.. _ref_func_char_length:

``char_length``
---------------

::

    char_length(s)

Description
^^^^^^^^^^^

``char_length`` computes the number of characters in a string.

Parameter Types
^^^^^^^^^^^^^^^

``s``
    ``string``

Return Type
^^^^^^^^^^^

``int``

Examples
^^^^^^^^

+-------------------------+----------+
| Function Call           | Result   |
+=========================+==========+
| ``char_length("über")`` | ``4``    |
+-------------------------+----------+



.. _ref_func_concat:

``concat``
----------

::

    concat(s [, ...])

Description
^^^^^^^^^^^

``concat`` concatenates all strings given as input arguments.
``NULL`` values are ignored, i.e., treated like an empty string.

Parameter Types
^^^^^^^^^^^^^^^

``s`` and all subsequent parameters
    ``string``

Return Type
^^^^^^^^^^^

``string``

Examples
^^^^^^^^

+-------------------------------+-------------+
| Function Call                 | Result      |
+===============================+=============+
| ``concat("abc", NULL, "22")`` | ``"abc22"`` |
+-------------------------------+-------------+



.. _ref_func_concat_ws:

``concat_ws``
-------------

::

    concat_ws(sep, s [, ...])

Description
^^^^^^^^^^^

``concat_ws`` concatenates all strings given as input arguments ``s`` using the separator ``sep``.
``NULL`` values are ignored.

Parameter Types
^^^^^^^^^^^^^^^

``sep``
    ``string``

``s`` and all subsequent parameters
    ``string``

Return Type
^^^^^^^^^^^

``string``

Examples
^^^^^^^^

+---------------------------------------+--------------+
| Function Call                         | Result       |
+=======================================+==============+
| ``concat_ws(":", "abc", NULL, "22")`` | ``"abc:22"`` |
+---------------------------------------+--------------+



.. _ref_func_format:

``format``
----------

::

    format(s, [x, ...])

Description
^^^^^^^^^^^

``format`` formats a variable number of arguments ``x`` according to a format string ``s``.

See the `Go package fmt <https://golang.org/pkg/fmt/>`_ for details of what formatting codes are allowed.

Parameter Types
^^^^^^^^^^^^^^^

``s``
    ``string``

``x`` and all subsequent parameters (optional)
    any

Return Type
^^^^^^^^^^^

``string``

Examples
^^^^^^^^

+--------------------------------+--------------+
| Function Call                  | Result       |
+================================+==============+
| ``format("%s-%d", "abc", 22)`` | ``"abc-22"`` |
+--------------------------------+--------------+



.. _ref_func_lower:

``lower``
---------

::

    lower(s)

Description
^^^^^^^^^^^

``lower`` converts a string ``s`` to lower case.
Non-ASCII Unicode characters are mapped to their lower case, too.

Parameter Types
^^^^^^^^^^^^^^^

``s``
    ``string``

Return Type
^^^^^^^^^^^

``string``

Examples
^^^^^^^^

+-------------------+------------+
| Function Call     | Result     |
+===================+============+
| ``lower("ÜBer")`` | ``"über"`` |
+-------------------+------------+




.. _ref_func_ltrim:

``ltrim``
---------

::

    ltrim(s)
    ltrim(s, chars)

Description
^^^^^^^^^^^

``ltrim`` removes the longest string consisting only of characters in ``chars`` (default: whitespace) from the start of ``s``.

Parameter Types
^^^^^^^^^^^^^^^

``s``
    ``string``

``chars`` (optional)
    ``string``

Return Type
^^^^^^^^^^^

``string``

Examples
^^^^^^^^

+-------------------------------+---------------+
| Function Call                 | Result        |
+===============================+===============+
| ``ltrim("  trim  ")``         | ``"trim  "``  |
+-------------------------------+---------------+
| ``ltrim("xyxtrimyyx", "xy")`` | ``"trimyyx"`` |
+-------------------------------+---------------+



.. _ref_func_md5:

``md5``
-------

::

    md5(s)

Description
^^^^^^^^^^^

``md5`` computes the MD5 checksum of a string ``s`` and returns it in hexadecimal format.

Parameter Types
^^^^^^^^^^^^^^^

``s``
    ``string``

Return Type
^^^^^^^^^^^

``string``

Examples
^^^^^^^^

+----------------+----------------------------------------+
| Function Call  | Result                                 |
+================+========================================+
| ``md5("abc")`` | ``"900150983cd24fb0d6963f7d28e17f72"`` |
+----------------+----------------------------------------+



.. _ref_func_octet_length:

``octet_length``
----------------

::

    octet_length(s)

Description
^^^^^^^^^^^

``octet_length`` computes the number of bytes in a string ``s``.
Note that due to UTF-8 encoding, this may differ from the number returned by ``char_length``.

Parameter Types
^^^^^^^^^^^^^^^

``s``
    ``string``

Return Type
^^^^^^^^^^^

``int``

Examples
^^^^^^^^

+--------------------------+----------+
| Function Call            | Result   |
+==========================+==========+
| ``octet_length("über")`` | ``5``    |
+--------------------------+----------+



.. _ref_func_overlay:

``overlay``
-----------

::

    overlay(s, repl, from)
    overlay(s, repl, from, for)

Description
^^^^^^^^^^^

``overlay`` replaces ``for`` characters in a string ``s`` with the string ``repl``, starting at ``from`` (1-based counting).
If ``for`` is not given, the length of ``repl`` is used as a default.

Parameter Types
^^^^^^^^^^^^^^^

``s``
    ``string``

``repl``
    ``string``

``from``
    ``int``

``for`` (optional)
    ``int``

Return Type
^^^^^^^^^^^

``string``

Examples
^^^^^^^^

+-------------------------------------+---------------+
| Function Call                       | Result        |
+=====================================+===============+
| ``overlay("Txxxxas", "hom", 2)``    | ``"Thomxas"`` |
+-------------------------------------+---------------+
| ``overlay("Txxxxas", "hom", 2, 4)`` | ``"Thomas"``  |
+-------------------------------------+---------------+




.. _ref_func_rtrim:

``rtrim``
---------

::

    rtrim(s)
    rtrim(s, chars)

Description
^^^^^^^^^^^

``rtrim`` removes the longest string consisting only of characters in ``chars`` (default: whitespace) from the end of ``s``.

Parameter Types
^^^^^^^^^^^^^^^

``s``
    ``string``

``chars`` (optional)
    ``string``

Return Type
^^^^^^^^^^^

``string``

Examples
^^^^^^^^

+-------------------------------+---------------+
| Function Call                 | Result        |
+===============================+===============+
| ``rtrim("  trim  ")``         | ``"  trim"``  |
+-------------------------------+---------------+
| ``rtrim("xyxtrimyyx", "xy")`` | ``"xyxtrim"`` |
+-------------------------------+---------------+



.. _ref_func_sha1:

``sha1``
--------

::

    sha1(s)

Description
^^^^^^^^^^^

``sha1`` computes the SHA1 checksum of a string ``s`` and returns it in hexadecimal format.

Parameter Types
^^^^^^^^^^^^^^^

``s``
    ``string``

Return Type
^^^^^^^^^^^

``string``

Examples
^^^^^^^^

+-----------------+------------------------------------------------+
| Function Call   | Result                                         |
+=================+================================================+
| ``sha1("abc")`` | ``"a9993e364706816aba3e25717850c26c9cd0d89d"`` |
+-----------------+------------------------------------------------+



.. _ref_func_sha256:

``sha256``
----------

::

    sha256(s)

Description
^^^^^^^^^^^

``sha256`` computes the SHA256 checksum of a string ``s`` and returns it in hexadecimal format.

Parameter Types
^^^^^^^^^^^^^^^

``s``
    ``string``

Return Type
^^^^^^^^^^^

``string``

Examples
^^^^^^^^

+-------------------+------------------------------------------------------------------------+
| Function Call     | Result                                                                 |
+===================+========================================================================+
| ``sha256("abc")`` | ``"ba7816bf8f01cfea414140de5dae2223b00361a396177a9cb410ff61f20015ad"`` |
+-------------------+------------------------------------------------------------------------+




.. _ref_func_strpos:

``strpos``
----------

::

    strpos(s, t)

Description
^^^^^^^^^^^

``strpos`` returns the index of the first occurence of ``t`` in ``s`` (1-based) or 0 if it is not found.

Parameter Types
^^^^^^^^^^^^^^^

``s``
    ``string``

``t``
    ``string``

Return Type
^^^^^^^^^^^

``int``

Examples
^^^^^^^^

+--------------------------+----------+
| Function Call            | Result   |
+==========================+==========+
| ``strpos("high", "ig")`` | ``2``    |
+--------------------------+----------+




.. _ref_func_substring:

``substring``
-------------

::

    substring(s, r)
    substring(s, from)
    substring(s, from, for)

Description
^^^^^^^^^^^

``substring(s, r)`` extracts the substring matching regular expression ``r`` from ``s``.
See the `Go regexp package <https://golang.org/pkg/regexp/>`_ for details of matching.

``substring(s, from, for)`` returns the ``for`` characters of ``str`` starting from the ``from`` index (1-based).
If ``for`` is not given, everything until the end of ``str`` is returned.

Which of those behaviors is used depends on the type of the second parameter (``int`` or ``string``).

Parameter Types
^^^^^^^^^^^^^^^

``s``
    ``string``

``r``
    ``string``

``from``
    ``int``

``for`` (optional)
    ``int``

Return Type
^^^^^^^^^^^

``string``

Examples
^^^^^^^^

+---------------------------------+-------------+
| Function Call                   | Result      |
+=================================+=============+
| ``substring("Thomas", "...$")`` | ``"mas"``   |
+---------------------------------+-------------+
| ``substring("Thomas", 2)``      | ``"homas"`` |
+---------------------------------+-------------+
| ``substring("Thomas", 2, 3)``   | ``"hom"``   |
+---------------------------------+-------------+





.. _ref_func_upper:

``upper``
---------

::

    upper(s)

Description
^^^^^^^^^^^

``upper`` converts a string ``s`` to upper case.
Non-ASCII Unicode characters are mapped to their upper case, too.

Parameter Types
^^^^^^^^^^^^^^^

``s``
    ``string``

Return Type
^^^^^^^^^^^

``string``

Examples
^^^^^^^^

+-------------------+------------+
| Function Call     | Result     |
+===================+============+
| ``upper("ÜBer")`` | ``"ÜBER"`` |
+-------------------+------------+



.. _ref_func_distance_us:

``distance_us``
---------------

::

    distance_us(u, v)

Description
^^^^^^^^^^^

``distance_us`` computes the signed temporal distance from ``u`` to ``v`` in microseconds.

Parameter Types
^^^^^^^^^^^^^^^

``u``
    ``timestamp``

``v``
    ``timestamp``

Return Type
^^^^^^^^^^^

``int``

Examples
^^^^^^^^

+-----------------------------------------------------------------------------------------------+--------------+
| Function Call                                                                                 | Result       |
+===============================================================================================+==============+
| ``distance_us("2016-02-09T05:40:25.123Z"::timestamp, "2016-02-09T05:41:25.456Z"::timestamp)`` | ``60333000`` |
+-----------------------------------------------------------------------------------------------+--------------+
| ``distance_us(clock_timestamp(), clock_timestamp())``                                         | ``2``        |
+-----------------------------------------------------------------------------------------------+--------------+



.. _ref_func_clock_timestamp:

``clock_timestamp``
-------------------

::

    clock_timestamp()

Description
^^^^^^^^^^^

``clock_timestamp`` returns the current date and time in UTC.

Return Type
^^^^^^^^^^^

``timestamp``



.. _ref_func_now:

``now``
-------

::

    now()

Description
^^^^^^^^^^^

``now`` returns the date and time in UTC of the point in time when processing of the current tuple started.
In particular and as opposed to ``clock_timestamp``, the timestamp returned by ``now()`` does not change during a processing run triggered by the arrival of a tuple.
For example, in

::

    SELECT RSTREAM clock_timestamp() AS a, clock_timestamp() AS b,
        now() AS c, now() AS d FROM ...

the values of ``a`` and ``b`` are most probably different by a very short timespan, but ``c`` and ``d`` are equal by definition of ``now()``.

``now`` cannot be used in an ``EVAL`` statement outside of a stream processing context.

Return Type
^^^^^^^^^^^

``timestamp``





.. _ref_func_coalesce:

``coalesce``
------------

::

    coalesce(x [, ...])

Description
^^^^^^^^^^^

``coalesce`` returns the first non-null input parameter or ``NULL`` if there is no such parameter.

Parameter Types
^^^^^^^^^^^^^^^

``x`` and all subsequent
    any

Return Type
^^^^^^^^^^^

same as input

Examples
^^^^^^^^

+-------------------------------+----------+
| Function Call                 | Result   |
+===============================+==========+
| ``coalesce(NULL, 17, "foo")`` | ``17``   |
+-------------------------------+----------+




Aggregate Functions Reference
=============================


.. _ref_func_array_agg:

``array_agg``
-------------

::

    array_agg(x)

Description
^^^^^^^^^^^

``array_agg`` returns an array containing all input values, including ``NULL`` values.
There is no guarantee on the order of items in the result.
Use the ``ORDER BY`` clause to achieve a certain ordering.

Parameter Types
^^^^^^^^^^^^^^^

``x``
    any

Return Type
^^^^^^^^^^^

``array``



.. _ref_func_avg:

``avg``
-------

::

    avg(x)

Description
^^^^^^^^^^^

``avg`` computes the average (arithmetic mean) of all input values.

Parameter Types
^^^^^^^^^^^^^^^

``x``
    ``int`` or ``float`` (mixed types are allowed)

Return Type
^^^^^^^^^^^

``float``



.. _ref_func_bool_and:

``bool_and``
------------

::

    bool_and(x)

Description
^^^^^^^^^^^

``bool_and`` returns ``true`` if all input values are true, otherwise ``false``.

Parameter Types
^^^^^^^^^^^^^^^

``x``
    ``bool``

Return Type
^^^^^^^^^^^

``bool``



.. _ref_func_bool_or:

``bool_or``
-----------

::

    bool_or(x)

Description
^^^^^^^^^^^

``bool_or`` returns ``true`` if at least one input value is true, otherwise ``false``.

Parameter Types
^^^^^^^^^^^^^^^

``x``
    ``bool``

Return Type
^^^^^^^^^^^

``bool``




.. _ref_func_count:

``count``
---------

::

    count(x)
    count(*)

Description
^^^^^^^^^^^

``count`` returns the number of input rows for which ``x`` is not ``NULL``, or the number of total rows if ``*`` is passed.

Parameter Types
^^^^^^^^^^^^^^^

``x``
    any

Return Type
^^^^^^^^^^^

``int``





.. _ref_func_json_object_agg:

``json_object_agg``
-------------------

::

    json_object_agg(k, v)

Description
^^^^^^^^^^^

``json_object_agg`` aggregates pairs of key ``k`` and value ``v`` as a map.
If both key and value are ``NULL``, the pair is ignored.
If only the value is ``NULL``, it is still added with the corresponding key. 
It is an error if only the key is ``NULL``.
It is an error if a key appears multiple times.

A map does not have an ordering, therefore there is no guarantee on the result map ordering, whether or not ``ORDER BY`` is used.


Parameter Types
^^^^^^^^^^^^^^^

``k``
    ``string``

``v``
    any

Return Type
^^^^^^^^^^^

``map``




.. _ref_func_max:

``max``
-------

::

    max(x)

Description
^^^^^^^^^^^

``max`` computes the maximum value of all input values.

Parameter Types
^^^^^^^^^^^^^^^

``x``
    ``int`` or ``float`` (mixed types are allowed)

Return Type
^^^^^^^^^^^

same as largest input value




.. _ref_func_median:

``median``
----------

::

    median(x)

Description
^^^^^^^^^^^

``median`` computes the median of all input values.

Parameter Types
^^^^^^^^^^^^^^^

``x``
    ``int`` or ``float`` (mixed types are allowed)

Return Type
^^^^^^^^^^^

``float``



.. _ref_func_min:

``min``
-------

::

    min(x)

Description
^^^^^^^^^^^

``min`` computes the minimum value of all input values.

Parameter Types
^^^^^^^^^^^^^^^

``x``
    ``int`` or ``float`` (mixed types are allowed)

Return Type
^^^^^^^^^^^

same as smallest input value



.. _ref_func_string_agg:

``string_agg``
--------------

::

    string_agg(x, sep)

Description
^^^^^^^^^^^

``string_agg`` returns a string with all values of ``x`` concatenated, separated by the (non-aggregate) ``sep`` parameter.

Parameter Types
^^^^^^^^^^^^^^^

``x``
    ``string``

``sep``
    ``string`` (scalar)

Return Type
^^^^^^^^^^^

``string``




.. _ref_func_sum:

``sum``
-------

::

    sum(x)

Description
^^^^^^^^^^^

``sum`` computes the sum of all input values.

Parameter Types
^^^^^^^^^^^^^^^

``x``
    ``int`` or ``float`` (mixed types are allowed)

Return Type
^^^^^^^^^^^

``float`` if the input contains a ``float``, ``int`` otherwise


