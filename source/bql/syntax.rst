**********
BQL Syntax
**********

Lexical Structure
=================

BQL has been designed to be easy to learn for people who have used SQL before.
While key words and commands differ in many cases, the basic structure, set of tokens, operators etc. is the same.
For example, the following is (syntactically) valid BQL input::

    SELECT RSTREAM given_name, last_name FROM persons [RANGE 1 TUPLES] WHERE age > 20;

    CREATE SOURCE s TYPE fluentd WITH host="example.com", port=12345;

    INSERT INTO file FROM data;

This is a sequence of three commands, one per line (although this is not required; more than one command can be on a line, and commands can usefully be split across lines).
Additionally, comments can occur in BQL input.
They are effectively equivalent to whitespace.

The type of commands that can be used in BQL is described in `Input/Output/State Definition`_ and `Queries`_.


Identifiers and Key Words
-------------------------

Tokens such as ``SELECT``, ``CREATE``, or ``INTO`` in the example above are examples of *key words*, that is, words that have a fixed meaning in the BQL language.
The tokens ``persons`` and ``file`` are examples of identifiers.
They identify names of streams, sources, or other objects, depending on the command they are used in.
Therefore they are sometimes simply called "names".
Key words and identifiers have the same lexical structure, meaning that one cannot know whether a token is an identifier or a key word without knowing the language.

BQL identifiers and key words must begin with a letter (``a-z``).
Subsequent characters can be letters, underscores, or digits (``0-9``).
Key words and unquoted identifiers are in general case insensitive.

However, there is one important difference between SQL and BQL when it comes to "column identifiers".
In BQL, there are no "columns" with names that the user can pick herself, but "field selectors" that describe the path to a value in a JSON-like document imported from outside the system.
Therefore field selectors are case-sensitive (in order to be able to deal with input of the form ``{"a": 1, "A": 2}``) and also there is a form that allows to use special characters; see `Field Selectors`_ for details.

.. note::

   At the moment, there is no restriction on the set of words that can be used as identifiers.
   However, it is strongly recommended not to use identifiers that are also key words in order to avoid confusion.
   Also, such restrictions on identifiers are likely to be introduced in future versions.


Constants
---------

There are multiple kinds of implicitly-typed constants in BQL: strings, decimal numbers (with and without fractional part) and booleans.
Constants can also be specified with explicit types, which can enable more accurate representation and more efficient handling by the system.
These alternatives are discussed in the following subsections.


String Constants
^^^^^^^^^^^^^^^^
A string constant in BQL is an arbitrary sequence of characters bounded by double quotes (``"``), for example ``"This is a string"``.
To include a double-quote character within a string constant, write two adjacent double quotes, e.g., ``"Dianne""s horse"``.

No escaping for special characters is supported at the moment, but any valid UTF-8 encoded byte sequence can be used.
See :ref:`the string data type reference<type_string>` for details.


Numeric Constants
^^^^^^^^^^^^^^^^^

There are two different numeric data types in BQL, ``int`` and ``float``, representing decimal numbers without and with fractional part, respectively.

An ``int`` constant is written as

::

    [-]digits

A ``float`` constant is written as

::

    [-]digits.digits

Scientific notation (``1e+10``) as well as Infinity and NaN cannot be used in BQL statements.

Some example of valid numerical constants::

    42
    3.5
    -36

See the type references for :ref:`type_int` and :ref:`type_float` for details.

.. note::

   For a number of operations/functions it makes a difference whether ``int`` or ``float`` is used (e.g., ``2/3`` is ``0``, but ``2.0/3`` is ``0.666666``).
   Be aware of that when writing constants in BQL statements.


Boolean Constants
^^^^^^^^^^^^^^^^^

There are two keywords for the two possible boolean values, namely ``true`` and ``false``.

See :ref:`the bool data type reference<type_bool>` for details.


Operators
---------

An operator is a sequence of the items from the following list::

    +
    -
    *
    /
    <
    >
    =
    !
    %

See the :ref:`chapter on Operators<bql_operators>` for the complete list of operators in BQL.
There are no user-defined operators at the moment.


Special Characters
------------------

Some characters that are not alphanumeric have a special meaning that is different from being an operator.
Details on the usage can be found at the location where the respective syntax element is described.
This section only exists to advise the existence and summarize the purposes of these characters.

- Parentheses (``()``) have their usual meaning to group expressions and enforce precedence.
  In some cases parentheses are required as part of the fixed syntax of a particular SQL command.
- Brackets (``[]``) are used in `Array Constructors`_ and in `Field Selectors`_, as well as in `Stream-to-Relation Operators`_.
- Curly brackets (``{}``) are used in `Map Constructors`_
- Commas (``,``) are used in some syntactical constructs to separate the elements of a list.
- The semicolon (``;``) terminates a BQL command.
  It cannot appear anywhere within a command, except within a string constant or quoted identifier.
- The colon (``:``) is used to separate stream names and field selectors, and within field selectors to select array slices (see `Extended Descend Operators`_).
- The asterisk (``*``) is used in some contexts to denote all the fields of a table row (see `Notes on Wildcards`_).
  It also has a special meaning when used as the argument of an aggregate function, namely that the aggregate does not require any explicit parameter.
- The period (``.``) is used in numeric constants and to denote descend in field selectors.


Comments
--------

A comment is a sequence of characters beginning with double dashes and extending to the end of the line, e.g.::

    -- This is a standard BQL comment

C-style comments cannot be used.


Operator Precedence
-------------------

The following table shows the operator precedence in BQL:

=============================================  =========================================
Operator/Element                               Description
=============================================  =========================================
``::``                                         typecast
``-``                                          unary minus
``*`` ``/`` ``%``                              multiplication, division, modulo
``+`` ``-``                                    addition, subtraction
``IS``                                         ``IS NULL`` etc.
(any other operator)                           e.g., ``||``
``=`` ``!=`` ``<>`` ``<=`` ``<`` ``>=`` ``>``  comparison operator
``NOT``                                        logical negation
``AND``                                        logical conjunction
``OR``                                         logical disjunction
=============================================  =========================================


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

The first option was already discussed in `Constants`_.
The following sections discuss the remaining options.


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

      ["child_key"]

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

For example, ``ids[1]`` in the document given above would return ``17``, ``dists[-2].other`` would return ``foo`` and just ``dists`` would return the array ``[{"other": "foo", "value": 7}, {"other": "bar", "value": 3.5}]``.

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
``nantoka["x"]``                   ``"y"``
``foo[0].bar``                     ``5``
``foo[0].hoge[-1].a``              ``3``
``["foo"][0]["hoge"][-1]["a"]``    ``3``
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

See the section :ref:`bql_operators` for details.


Function Calls
--------------

The syntax for a function call is the name of a function, followed by its argument list enclosed in parentheses::

    function_name([expression [, expression ... ]])

For example, the following computes the square root of 2::

    sqrt(2);

The list of built-in functions is described in section `Functions`_.

.. _bql_syntax_aggregates:

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

    string_agg(name, ", ")

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

    string_agg(first_name || " " || last_name, "," ORDER BY last_name)

will create a comma-separated list of names, ordered ascending by the last name.

See `Aggregate Functions`_ for a list of built-in aggregate functions.


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

    SELECT RSTREAM [7, 2 * stream:a, true, "blue"] FROM ...

Each element of the array can have a different type.
In particular, the wildcard is also allowed as an expression and will include the whole current row as an array element.

.. note::

   Single-element arrays of strings could also be interpreted as JSON Paths and are therefore required to have a trailing comma after their only element: ``["foo",]``


Map Constructors
----------------

A map constructor is an expression that builds a map value using string keys and arbitrary values for its member elements.
A simple map constructor consists of a left curly bracket ``{``, a list of ``"key": value`` pairs (separated by commas) for the map elements, and finally a right curly bracket ``}``.
For example::

    SELECT RSTREAM {"a_const": 7, "prod": 2 * stream:a} FROM ...

The keys must be string literals (i.e., they can not be computed expressions); in particular they must be written using double quotes.
The values can be arbitrary expressions, including a wildcard.


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

BQL allows functions to be called using only the positional notation.
In positional notation, a function call is written with its argument values in the same order as they are defined in the function declaration.
Therefore, while some parameters of a function can be optional, these parameters can only be omitted *at the end* of the parameter list.

For example,

::

    log(100)
    log(100, 2)

are both valid function calls computing the logarithm of a function.
The first one uses the default value 10 for the logarithm base, the second one uses the given value 2.

