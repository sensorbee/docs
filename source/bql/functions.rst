*********
Functions
*********

BQL provides a number of built-in functions that are described in this chapter.
Function names and meaning of parameters have been heavily inspired by `PostgreSQL <http://www.postgresql.org/docs/9.5/static/functions.html>`_.
However, be aware that the accepted and returned types may differ as there is no simple mapping between BQL and SQL data types.

Numeric Functions
=================

General Functions
-----------------

The table below shows some common mathematical functions that can be used in BQL.

+----------------------------------+--------------------------------------------+
| Function                         | Description                                |
+==================================+============================================+
| ``abs(x)``                       | absolute value                             |
+----------------------------------+--------------------------------------------+
| ``cbrt(x)``                      | cube root                                  |
+----------------------------------+--------------------------------------------+
| ``ceil(x)``                      | round up to nearest integer                |
+----------------------------------+--------------------------------------------+
| ``degrees(x)``                   | radians to degrees                         |
+----------------------------------+--------------------------------------------+
| ``div(y, x)``                    | integer quotient of ``y``/``x``            |
+----------------------------------+--------------------------------------------+
| ``exp(x)``                       | exponential                                |
+----------------------------------+--------------------------------------------+
| ``floor(x)``                     | round down to nearest integer              |
+----------------------------------+--------------------------------------------+
| ``ln(x)``                        | natural logarithm                          |
+----------------------------------+--------------------------------------------+
| ``log(x)``                       | base 10 logarithm                          |
+----------------------------------+--------------------------------------------+
| ``log(b, x)``                    | logarithm to base b                        |
+----------------------------------+--------------------------------------------+
| ``mod(y, x)``                    | remainder of ``y``/``x``                   |
+----------------------------------+--------------------------------------------+
| ``pi()``                         | "π" constant                               |
+----------------------------------+--------------------------------------------+
| ``power(a, b)``                  | ``a`` raised to the power of ``b``         |
+----------------------------------+--------------------------------------------+
| ``radians(x)``                   | degrees to radians                         |
+----------------------------------+--------------------------------------------+
| ``round(x)``                     | round to nearest integer                   |
+----------------------------------+--------------------------------------------+
| ``sign(x)``                      | sign of the argument (-1, 0, +1)           |
+----------------------------------+--------------------------------------------+
| ``sqrt(x)``                      | square root                                |
+----------------------------------+--------------------------------------------+
| ``trunc(x)``                     | truncate toward zero                       |
+----------------------------------+--------------------------------------------+
| ``width_bucket(x, l, r, c)``     | bucket of ``x`` in a histogram             |
+----------------------------------+--------------------------------------------+

If a given parameter is outside the mathematically valid range for that function (e.g., ``sqrt(-2)``, ``log(0)``, ``div(2.0, 0.0)``) and the return type is ``float``, then ``NaN`` is returned.
However, if the return type is ``int`` (e.g., ``div(2, 0)``), there is no ``NaN`` option and an error will occur instead.


Pseudo-Random Functions
-----------------------

The table below shows functions for generating pseudo-random numbers.

+----------------+-----------------------------------------------------------------------+
| Function       | Description                                                           |
+================+=======================================================================+
| ``random()``   | random value in the range :math:`0.0 <= x < 1.0`                      |
+----------------+-----------------------------------------------------------------------+
| ``setseed(x)`` | set seed (:math:`-1.0 <= x <= 1.0`) for subsequent ``random()`` calls |
+----------------+-----------------------------------------------------------------------+

The characteristics of the values returned by ``random()`` are equal to those from `the Go rand module <https://golang.org/pkg/math/rand/>`_.
It is not suitable for cryptographic applications.


Trigonometric Functions
-----------------------

Finally, the table below shows the available trigonometric functions.
All trigonometric functions take arguments and return values of type ``float``.
Trigonometric functions arguments are expressed in radians.
Inverse functions return values are expressed in radians.


+-------------+-----------------+
| Function    | Description     |
+=============+=================+
| ``acos(x)`` | inverse cosine  |
+-------------+-----------------+
| ``asin(x)`` | inverse sine    |
+-------------+-----------------+
| ``atan(x)`` | inverse tangent |
+-------------+-----------------+
| ``cos(x)``  | cosine          |
+-------------+-----------------+
| ``cot(x)``  | cotangent       |
+-------------+-----------------+
| ``sin(x)``  | sine            |
+-------------+-----------------+
| ``tan(x)``  | tangent         |
+-------------+-----------------+


String Functions
================

The table below shows some common functions for strings that can be used in BQL.

+-------------------------------+---------------------------------------------------+
| Function                      | Description                                       |
+===============================+===================================================+
| ``bit_length(s)``             | number of bits in string                          |
+-------------------------------+---------------------------------------------------+
| ``btrim(s)``                  |   remove whitespace from the start/end of ``s``   |
+-------------------------------+---------------------------------------------------+
| ``btrim(s, chars)``           | remove ``chars`` from the start/end of ``s``      |
+-------------------------------+---------------------------------------------------+
| ``char_length(s)``            | number of characters in ``s``                     |
+-------------------------------+---------------------------------------------------+
| ``concat(s [, ...])``         | concatenate all arguments                         |
+-------------------------------+---------------------------------------------------+
| ``concat_ws(sep, s [, ...])`` | concatenate ``s`` arguments with separator        |
+-------------------------------+---------------------------------------------------+
| ``format(s, [x, ...])``       | format arguments using a format string            |
+-------------------------------+---------------------------------------------------+
| ``lower(s)``                  | convert ``s`` to lower case                       |
+-------------------------------+---------------------------------------------------+
| ``ltrim(s)``                  | remove whitespace from the start of ``s``         |
+-------------------------------+---------------------------------------------------+
| ``ltrim(s, chars)``           | remove ``chars`` from the start of ``s``          |
+-------------------------------+---------------------------------------------------+
| ``md5(s)``                    | MD5 hash of ``s``                                 |
+-------------------------------+---------------------------------------------------+
| ``octet_length(s)``           | number of bytes in ``s``                          |
+-------------------------------+---------------------------------------------------+
| ``overlay(s, r, from)``       | replace substring                                 |
+-------------------------------+---------------------------------------------------+
| ``overlay(s, r, from, for)``  | replace substring                                 |
+-------------------------------+---------------------------------------------------+
| ``rtrim(s)``                  | remove whitespace from the end of ``s``           |
+-------------------------------+---------------------------------------------------+
| ``rtrim(s, chars)``           | remove ``chars`` from the end of ``s``            |
+-------------------------------+---------------------------------------------------+
| ``sha1(s)``                   | SHA1 hash of ``s``                                |
+-------------------------------+---------------------------------------------------+
| ``sha256(s)``                 | SHA256 hash of ``s``                              |
+-------------------------------+---------------------------------------------------+
| ``strpos(s, t)``              | location of substring ``t`` in ``s``              |
+-------------------------------+---------------------------------------------------+
| ``substring(s, r)``           | extract substring matching regex ``r`` from ``s`` |
|                               |                                                   |
+-------------------------------+---------------------------------------------------+
| ``substring(s, from)``        | extract substring                                 |
+-------------------------------+---------------------------------------------------+
| ``substring(s, from, for)``   | extract substring                                 |
+-------------------------------+---------------------------------------------------+
| ``upper(s)``                  | convert ``s`` to upper case                       |
+-------------------------------+---------------------------------------------------+


Time Functions
==============

+-----------------------+--------------------------------------------------------------+
| Function              | Description                                                  |
+=======================+==============================================================+
| ``distance_us(u, v)`` | signed temporal distance from ``u`` to ``v`` in microseconds |
+-----------------------+--------------------------------------------------------------+
| ``clock_timestamp()`` | current date and time (changes during statement execution)   |
+-----------------------+--------------------------------------------------------------+
| ``now()``             | date and time when processing of current tuple was started   |
+-----------------------+--------------------------------------------------------------+


Other Scalar Functions
======================

+-------------------------+--------------------------------------------+
| Function                | Description                                |
+=========================+============================================+
| ``coalesce(x [, ...])`` | first non-null input parameter             |
+-------------------------+--------------------------------------------+


Aggregate Functions
===================

Aggregate functions compute a single result from a set of input values.
The built-in normal aggregate functions are listed in the table below.
The special syntax considerations for aggregate functions are explained in `Aggregate Expressions`_.

+---------------------------+---------------------------------------------------------------+
| Function                  | Description                                                   |
+===========================+===============================================================+
| ``array_agg(x)``          | input values, including nulls, concatenated into an array     |
+---------------------------+---------------------------------------------------------------+
| ``avg(x)``                | the average (arithmetic mean) of all input values             |
+---------------------------+---------------------------------------------------------------+
| ``bool_and(x)``           | true if all input values are true, otherwise false            |
+---------------------------+---------------------------------------------------------------+
| ``bool_or(x)``            | true if at least one input value is true, otherwise false     |
+---------------------------+---------------------------------------------------------------+
| ``count(x)``              | number of input rows for which ``x`` is not null              |
+---------------------------+---------------------------------------------------------------+
| ``count(*)``              | number of input rows                                          |
+---------------------------+---------------------------------------------------------------+
| ``json_object_agg(k, v)`` | aggregates name/value pairs as a map                          |
+---------------------------+---------------------------------------------------------------+
| ``max(x)``                | maximum value of ``x`` across all input values                |
+---------------------------+---------------------------------------------------------------+
| ``median(x)``             | the median of all input values                                |
+---------------------------+---------------------------------------------------------------+
| ``min(x)``                | minimum value of ``x`` across all input values                |
+---------------------------+---------------------------------------------------------------+
| ``string_agg(x, sep)``    | input values concatenated into a string, separated by ``sep`` |
+---------------------------+---------------------------------------------------------------+
| ``sum(x)``                | sum of ``x`` across all input values                          |
|                           |                                                               |
+---------------------------+---------------------------------------------------------------+

It should be noted that except for ``count``, these functions return a ``NULL`` value when no rows are selected.
In particular, ``sum`` of no rows returns ``NULL``, not zero as one might expect, and ``array_agg`` returns ``NULL`` rather than an empty array when there are no input rows.
The ``coalesce`` function can be used to substitute zero or an empty array for ``NULL`` when necessary.
