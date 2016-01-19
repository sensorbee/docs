**********
BQL Syntax
**********

Lexical Structure
=================

TODO


Value Expressions
=================

Value expressions are used in a variety of contexts, such as in the target list or filter condition of the ``SELECT`` command.
The expression syntax allows the calculation of values from primitive parts using arithmetic, logical, set, and other operations.

A value expression is one of the following:

- A constant or literal value
- A field selector
- A row metadata reference
- An operator invocation
- A function call
- An aggregate expression
- A type cast
- An array constructor
- A map constructor
- Another value expression in parentheses (used to group subexpressions and override precedence)


Literals
--------

TODO: reference either the previous subsection or the Types section


Field Selectors
---------------

In SQL, each table has a well-defined schema with columns, column names and column types.
Therefore, a column name is enough to check whether that column exists, what type it has and if the type that will be extracted matches the type expected by the surrounding expression.

In BQL, each row corresponds to a JSON-like object, i.e., a map with string keys and values that have one of several data types (see `Data Types and Conversions`_).
In particular, nested maps and arrays are commonplace in the data streams used with BQL.
For example, a row could look like::

    {"ids": [3, 17, 21, 5],
     "dists": [
      {"other": "foo", "value": 7},
      {"other": "bar", "value": 3.5}
     ],
     "found": true}

To deal with such nested data structures, BQL uses a subset of `JSON Path <http://goessner.net/articles/JsonPath/>`_ to address values in a row.

Basic Descend Operators
^^^^^^^^^^^^^^^^^^^^^^^

In general, a JSON path describes a path to descend in a JSON structure, starting from the top.
The basic rules are:

- If the current node is a map, then

  ::

      .child_key

  or

  ::

      ['child_key']

  mean "descend to the child node with the key ``child_key``".
  The second form must be used if the key name has a non-identifier shape (e.g., contains spaces, dots, brackets or similar).
  It is an error if the current node is not a map.
  It is an error if the current node does not have such a child node.
- If the current node is an array, then

  ::

      [k]

  means "descend to the (zero-based) :math:`k`-th element in the array".
  Negative indices count from end end of the array (as in Python).
  It is an error if the current node is not an array.
  It is an error if the given index is out of bounds.

The first element of a JSON Path must always be a "map access" component (since the document is always a map) and the leading dot must be omitted.

For example, ``ids[1]`` in the document given above would return ``17``, ``dists[-2].other`` would return ``7`` and just ``dists`` would return the array ``[{"other": "foo", "value": 7}, {"other": "bar", "value": 3.5}]``.

Extended Descend Operators
^^^^^^^^^^^^^^^^^^^^^^^^^^

There is limited support for array slicing and recursive descend:

- If the current node is a map or an array, then

  ::

      ..child_key

  returns an array of all values below the current node that have the key ``child_key``.
  However, if a node with key ``child_key`` has been found, it will be returned as is, even if it may possibly itself contain that key again.

  This selector cannot be used as the first component of a JSON Path.
  It is an error if the current node is not a map or an array.
  It is *not* an error if there is no child element with the given key.
- If the current node is an array, then

  ::

      [start:end]

  returns an array of all values with the indexes in the range :math:`[\text{start}, \text{end}-1]`.
  One or both of ``start`` and ``end`` can be omitted, meaning "from the first element" and "until the last element", respectively.

  ::

      [start:end:step]

  returns an array of all elements with the indexes :math:`[\text{start}, \text{start}+\text{step}, \text{start}+2\cdot\text{step}, \cdot\cdot\cdot, \text{end}-1]` if ``step`` is positive, or :math:`[\text{start}, \text{start}-\text{step}, \text{start}-2\cdot\text{step}, \cdot\cdot\cdot, \text{end}+1]` if it is negative.
  (This description is only true for positive indices, but in fact also negative indices can be used, again counting from the end of the array.)
  In general, the behavior has been implemented to be very close to Python's list slicing.

  These selectors cannot be used as the first component of a JSON Path.
  It is an error if it can be decided independent of the input data that the specified values do not make sense (e.g., ``step`` is 0, or ``end`` is larger than ``start`` but ``step`` is negative), but slices that will always be empty (e.g., ``[2:2]``) are valid.
  Also, if it depends on the input data whether a slice specification is valid or not (e.g., ``[4:-4]``) it is not an error, but an empty array is returned.
- If the slicing or recursive descend operators are followed by ordinary JSON Path operators as described before, their meaning changes to "... for every element in the array".
  For example, ``list[1:3].foo`` has the same result as ``[list[1].foo, list[2].foo, list[3].foo]`` (except that the latter would fail if ``list`` is not long enough) or a Python list comprehension such as ``[x.foo for x in list[1:3]]``.
  However, it is not possible to chain multiple list-returning operators: ``list[1:3]..foo`` or ``foo..bar..hoge`` are invalid.

Examples
^^^^^^^^

Given the input data

::

    {
        "foo": [
            {"hoge": [
                {"a": 1, "b": 2},
                {"a": 3, "b": 4} ],
             "bar": 5},
            {"hoge": [
                {"a": 5, "b": 6},
                {"a": 7, "b": 8} ],
             "bar": 2},
            {"hoge": [
                {"a": 9, "b": 10} ],
             "bar": 8}
        ],
        "nantoka": {"x": "y"}
    }

the following table is supposed to illustrate the effect of various JSON Path expressions.

=================================  ================
 Path                               Result
=================================  ================
``nantoka``                        ``{"x": "y"}``
``nantoka.x``                      ``"y"``
``nantoka['x']``                   ``"y"``
``foo[0].bar``                     ``5``
``foo[0].hoge[-1].a``              ``3``
``['foo'][0]['hoge'][-1]['a']``    ``3``
``foo[1:2].bar``                   ``[2, 8]``
``foo..bar``                       ``[5, 2, 8]``
``foo..hoge[0].b``                 ``[2, 6, 10]``
=================================  ================


Row Metadata References
-----------------------

Metadata is the data that is attached to a tuple but which cannot be accessed as part of the normal row data.
At the moment, the only metadata that can be accessed from within BQL is a tuple's system timestamp (the time that was set by the source that created it).
This timestamp can be accessed using the ``ts()`` function.
If multiple streams are joined, a stream prefix is required to identify the input tuple that is referred to, i.e.,

::

     stream_name:ts()


Operator Invocations
--------------------

There are three possible syntaxes for an operator invocation::

    expression  operator  expression

    operator  expression

    expression  operator

See the section `Operators`_ for details.


Function Calls
--------------

The syntax for a function call is the name of a function, followed by its argument list enclosed in parentheses::

    function_name([expression [, expression ... ]])

For example, the following computes the square root of 2::

    sqrt(2);

The list of built-in functions is described in section `Functions`_.


Aggregate Expressions
---------------------

An aggregate expression represents the application of an aggregate function across the rows selected by a query.
An aggregate function reduces multiple inputs to a single output value, such as the sum or average of the inputs.
The syntax of an aggregate expression is the following::

    function_name(expression [, ... ] [ order_by_clause ])

where ``function_name`` is a previously defined aggregate and expression is any value expression that does not itself contain an aggregate expression.
The optional ``order_by_clause`` is described below.

In BQL, aggregate functions can take aggregate and non-aggregate parameters.
For example, the ``string_agg`` function can be called like

::

    string_agg(name, ', ')

to return a comma-separated list of all names in the respective group.
However, the second parameter is not an aggregation parameter, so for a statement like

::

    SELECT RSTREAM string_agg(name, sep) FROM ...

``sep`` must be mentioned in the ``GROUP BY`` clause.

For many aggregate functions (e.g., ``sum`` or ``avg``), the order of items in the group does not matter.
However, for other functions (e.g., ``string_agg``) the user has certain expectations with respect to the order that items should be fed into the aggregate function.
In this case, the ``order_by_clause`` with the syntax

::

    ORDER BY expression [ASC | DESC] [ , expression [ASC | DESC] ... ]

can be used.
The rows that are fed into the aggregate function are sorted by the values of the given expression in ascending (default) or descending mode.
For example,

::

    string_agg(first_name || ' ' || last_name, ',' ORDER BY last_name)

will create a comma-separated list of names, ordered ascending by the last name.

See `TODO: Aggregate Functions`_ for a list of built-in aggregate functions.


Type Casts
----------

A type cast specifies a conversion from one data type to another.
BQL accepts two equivalent syntaxes for type casts::

    CAST(expression AS type)
    expression::type

When a cast is applied to a value expression, it represents a run-time type conversion.
The cast will succeed only if a suitable type conversion operation has been defined, see `Conversions`_.



Array Constructors
------------------

An array constructor is an expression that builds an array value using values for its member elements.
A simple array constructor consists of a left square bracket ``[``, a list of expressions (separated by commas) for the array element values, and finally a right square bracket ``]``.
For example::

    SELECT RSTREAM [7, 2 * stream:a, true, 'blue'] FROM ...

Each element of the array can have a different type.
In particular, the wildcard is also allowed as an expression and will include the whole current row as an array element.

.. note::

   Single-element arrays of strings could also be interpreted as JSON Paths and are therefore required to have a trailing comma after their only element: ``['foo',]``


Map Constructors
----------------

A map constructor is an expression that builds a map value using string keys and arbitrary values for its member elements.
A simple map constructor consists of a left curly bracket ``{``, a list of ``'key': value`` pairs (separated by commas) for the map elements, and finally a right curly bracket ``}``.
For example::

    SELECT RSTREAM {'a_const': 7, 'prod': 2 * stream:a} FROM ...

The keys must be string literals (i.e., they can not be computed expressions), but the values can be arbitrary expressions, including wildcard.


Expression Evaluation Rules
---------------------------

The order of evaluation of subexpressions is not defined.
In particular, the inputs of an operator or function are not necessarily evaluated left-to-right or in any other fixed order.

Furthermore, if the result of an expression can be determined by evaluating only some parts of it, then other subexpressions might not be evaluated at all.
For instance, if one wrote::

    true OR somefunc()

then ``somefunc()`` would (probably) not be called at all.
The same would be the case if one wrote::

    somefunc() OR true

Note that this is *not* the same as the left-to-right "short-circuiting" of Boolean operators that is found in some programming languages.


Calling Functions
=================

TODO
