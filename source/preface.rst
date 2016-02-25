#######
Preface
#######

This is the official documentation of SensorBee. It describes all the
functionality that the current version of SensorBee officially supports.

This document is structured as follows:

* Preface, this part, provides general information of SensorBee.
* :ref:`Part I <tutorial>` is an introduction for new users through some
  tutorials.
* :ref:`Part II <bql>` documents the syntax and specification of the BQL
  language.
* :ref:`Part III <server_programming>` describes information for advancedusers
  about extensibility capabilities of the server.
* :ref:`Reference <ref>` contains reference information about BQL statements,
  built-in components, and client programs.

******************
What is SensorBee?
******************

SensorBee is an open source lightweight stateful streaming data processing
engine. SensorBee is designed to be used for streaming ETL
(Extract/Transform/Load) at the edge of the network including
`Fog <http://www.cisco.com/c/dam/en_us/solutions/trends/iot/docs/computing-overview.pdf>`_.
In ETL operations, SensorBee mainly focuses on data transformation and data
enrichment, especially using machine learning. SensorBee is very small (< 30MB)
and runs on small computers such as Raspberry Pi.

The processing flow in SensorBee is written in BQL, a dialect of CQL
(Continuous Query Language), which is similar to SQL but extended for streaming
data processing. Its internal data structure (tuple) is compatbile to JSON
rather than rows in RDBMSs. Therefore, in addition to regular SQL expressions,
BQL implements JSON notation and type conversions that works well with JSON.
BQL is also schemaless at the moment to support rapid prototyping and
integration.

.. note::

    Supporting a schema in SensorBee is being planned to increase its
    robustness, debuggability, and speed. However, the version that will support
    the feature has not been decieded yet.

SensorBee manages user-defined states (UDSs) and BQL utilizes those states to
perform stateful processing on streaming data. An example of stateful processing
is machine learning. With the Python extension, SensorBee supports deep learning
using `Chainer <http://chainer.org/>`_, which is a flexible deep learning
framework also developed by Preferred Networks, Inc. and Preferred
Infrastructure, Inc.. The combination of SensorBee and Chainer enables users to
support not only online analysis but also online training of deep learning
models at the edge of the network with a help of GPUs. Preprocessing of data
and feature extraction from preprocessed results can be written in BQL. The
results can be computed in an online manner and directly connected to deep
learning models implemented with Chainer.

By combining JSON-like data structure of BQL and machine learning, SensorBee
becomes good at handling unstructured data such as text written in natural
languages and even video streams, which are not well supported by majority of
data processing engines. Therefore, SensorBee can be located, for example,
between a video camera and Cloud-based (semi-structured) data analytics
services so that those services don't have to analyze raw video images and
can only utilize the information extracted from them by SensorBee.

SensorBee can be extended to work with existing databases or data processing
solutions by developing data source or sink plugins. For example, it officially
provides plugins for `fluentd <http://www.fluentd.org/>`_, which is an open
source data collector and has various input and output plugins for major
databases and Cloud services.

SensorBee is **not** designed for:

* very large scale data processing
* massively parallel streaming data processing
* accurate numerical computation without any error

***********
Conventions
***********

The following conventions are used in the synopsis of a command:

* Brackets (``[`` and ``]``) indicate optional parts.

    * Some statements such as ``SELECT`` have ``[`` and ``]`` as a part of the
      statement. In that case, those brackets are enclosed with single quotes
      (``'``).

* Braces (``{`` and ``}``) and vertical lines (``|``) indicate that one of
  candidates in braces must be chosen (e.g. one of a, b, or c has to be selected
  ``{a | b | c}``).

* Dots (``...``) mean that the preceding element can be repeated.

Types and keywords in BQL are written with ``fixed-size fonts``.

*******************
Further Information
*******************

Besides this documentation, there're other resources about SensorBee:

Website

    TODO

github

    The `sensorbee <https://github.com/sensorbee>`_ organization has the core
    source code of SensorBee and its plugins.

Godoc

    SensorBee is written in Go and the document of its source code can be found
    at (TODO: godoc)

Mailing lists

    TODO

