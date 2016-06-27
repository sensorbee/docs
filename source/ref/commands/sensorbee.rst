.. _ref_commands_sensorbee:

sensorbee
=========

``sensorbee`` is the main command to manipulate SensorBee. ``sensorbee``
consists of a set of following subcommands:

* :ref:`ref_commands_sensorbee_run`
* :ref:`ref_commands_sensorbee_runfile`
* :ref:`ref_commands_sensorbee_shell`
* :ref:`ref_commands_sensorbee_topology`

``sensorbee`` command can needs to be created by `build_sensorbee` command and
all the example commands are written as ``./sensorbee`` to emphasize that
there's no default ``sensorbee`` command.

See each command's reference for details.

Flags and Options
-----------------

``--help`` or ``-h``

    When this flag is given, the command prints the usage of itself.

``--version`` or ``-v``

    The command prints the version of SensorBee.

.. _ref_commands_sensorbee_run:

sensorbee run
=============

``sensorbee run`` runs the SensorBee server that manages multiple topologies
that can dynamically modified at runtime.

Basic Usage
-----------

::

    $ ./sensorbee run -c sensorbee.yaml

.. _ref_commands_sensorbee_run_config:

Configuration
-------------

A configuration file can optionally be provided for ``sensorbee run`` command.
The file is written in `YAML <http://yaml.org/>`_ and has following optional
sections:

* ``logging``
* ``network``
* ``storage``
* ``topologies``

``logging``
^^^^^^^^^^^

The ``logging`` section customizes behavior of the logger. It has following
optional parameters:

``target``

    The ``target`` parameter changes the destination of log messages. Its value
    can be of followings:

    * ``stdout``: write log messages to stdout
    * ``stderr``: write log messages to stderr
    * file path: a path to the log file

    When the value is neither ``stdout`` nor ``stderr``, it's considered to be
    a file path. The default value of this parameter is ``stderr``.

``min_log_level``

    This option specifies the minimum level (severity) of log messages to be
    written. Valid values are one of ``debug``, ``info``, ``warn``, ``warning``,
    ``error``, or ``fatal``. ``warn`` can also be ``warning``. When ``debug`` is
    given, all levels of messages will be written into the log. When the value
    is ``error``, only log messages with ``error`` or ``fatal`` level will be
    written. The default value of this parameter is ``info``.

``log_dropped_tuples``

    The SensorBee server can prints a log message and contents of tuples when
    they're dropped from a topology. When this option is ``true``, the server
    writes log messages reporting dropped tuples. When it's ``false``, the
    server doesn't. The default value of this option is ``false``.

``summarize_dropped_tuples``

    This option turns on or off summarization of dropped tuple logging. Valid
    values for this option is ``true`` or ``false``. When its value is the
    ``true``, dropped tuples are summarized in log messages.

    .. note::

        At the current version, only ``blob`` fields are summarized to
        ``"(blob)"``. Other configuration parameters will be supported in the
        future version such as the maximum number of fields, the maximum depths
        of maps, the maximum length of arrays, and so on.

    When ``false`` is specified, each log message shows a complete JSON that
    are compatible to the original tuple. Although this is useful for debugging,
    tuples containing large binary data like images may result in disk.

    The default value of this option is ``false``.

Example::

    logging:
      target: /path/to/sensorbee.log
      min_log_level: info
      log_dropped_tuples: true
      summarize_dropped_tuples: true

``network``
^^^^^^^^^^^

The ``network`` section has parameters related to server's network
configuration. It has following optional parameters:

``listen_on``

    This parameter controls how the server expose its listening port. The syntax
    of the value is like ``host:port``. ``host`` can be IP addresses such as
    ``0.0.0.0`` or ``127.0.0.1``. When ``host`` is given, the server only
    listens on the interface with the given host address. If the ``host`` is
    omitted, the server listens on all available interfaces, that is, the server
    accepts connections from any host. The default value of this parameter is
    ``:15601``.

Example::

    network:
      listen_on: ":15601"

``storage``
^^^^^^^^^^^

The ``storage`` section contains the configuration of storages used for saving
UDSs or other information. It has following optional subsections:

* ``uds``

``uds``
"""""""

The ``uds`` subsection configures the storage for saving and loading UDSs. It
provides following optional parameters:

``type``

    The type name of the storage. ``in_memory`` is used as the default value.

``params``

    ``params`` has subparameter specific to the given ``type``.

Currently, following types are available:

* ``in_memory``
* ``fs``

Descriptions of types and parameters are provided below:

``in_memory``

    ``in_memory`` saves UDSs in memory. It loses all saved data when the server
    restarts. This type doesn't have any parameter.

    Example::

        storage:
          uds:
            type: in_memory

``fs``

    ``fs`` saves UDSs in the local file system. It has following required
    parameters:

    ``dir``

        ``dir`` has the path to the directory that saved data will be stored.

    ``fs`` also has following optional parameters:

    ``temp_dir``

        ``temp_dir`` has the path to the temporary directory that is used when
        the UDS writes data. After the UDS has written all the data, the file
        is move to the directory specified by ``dir`` parameter. The same value
        as ``dir`` is used by default.

    The file name of each saved UDS is formatted as
    ``<topology>-<name>-<tag>.state``.

    Example::

        storage:
          uds:
            type: fs
            params:
              dir: /path/to/uds_dir
              temp_dir: /tmp

``topologies``
^^^^^^^^^^^^^^

The ``topologies`` section contains the configuration of topologies in the
following format::

    topologies:
      name_of_topology1:
        ... configuration for name_of_topology1 ...
      name_of_topology2:
        ... configuration for name_of_topology2 ...
      name_of_topology3:
        ... configuration for name_of_topology3 ...
      ... other topologies ...

Topologies listed in this section will be created at the startup of the server
based on the sub-configuration of each topology. Following optional
configuration parameters are provided for each topology:

``bql_file``

    This parameter has the path to the file containing BQL statements for the
    topology. All statements are executed before the server gets ready. If the
    execution fails, the server would exit with an error.

Example::

    $ ls
    my_topology.bql
    sensorbee.yaml
    $ cat my_topology.bql
    CREATE SOURCE fluentd TYPE fluentd;
    CREATE STREAM users AS
        SELECT RSTREAM users FROM fluentd [RANGE 1 TUPLES];
    CREATE SINK user_file TYPE file WITH path = "users.jsonl";
    $ cat sensorbee.yaml
    topologies:
      my_topology:
        bql_file: my_topology.bql
    $ ./sensorbee run -c sensorbee.yaml

As a result of these commands above, the server started with ``sensorbee.yaml``
has a topology named ``my_topology``. The topology has three nodes: ``fluentd``,
``users``, and ``user_file``.

.. note::

    This is the only way to persist the configuration of topologies at the
    moment. Any updates applied at runtime will not be reflected into the bql file.
    For example, if the server restarts after creating a new stream in
    ``my_topology``, the new stream will be lost unless it's explicitly added
    to ``my_topology.bql`` manually.

The configuration of a topology can be empty::

    topologies:
      my_empty_topology:

In this case, an empty topology ``my_empty_topology`` will be created so that
the ``sensorbee topology create`` command doesn't have to be executed every
time the server restarts.

A Complete Example
^^^^^^^^^^^^^^^^^^

::

    logging:
      target: /path/to/sensorbee.log
      min_log_level: info
      log_dropped_tuples: true
      summarize_dropped_tuples: true

    network:
      listen_on: ":15601"

    storage:
      uds:
        type: fs
        params:
          dir: /path/to/uds_dir
          temp_dir: /tmp

    topologies:
      empty_topology:
      my_topology:
        bql_file: /path/to/my_topology.bql

Flags and Options
-----------------

``--config path`` or ``-c path``

    This option receives the path of the configuration file. By default, the
    value is empty and no configuration file is used. This value can also be
    passed through ``SENSORBEE_CONFIG`` environment variable.

``--help`` or ``-h``

    When this flag is given, the command prints the usage of itself.

.. _ref_commands_sensorbee_runfile:

sensorbee runfile
=================

``sensorbee runfile`` runs a single BQL file. This command is mainly designed
for offline data processing but can be used as a standalone SensorBee process
that doesn't expose any interface to manipulate the topology.

``sensorbee runfile`` stops after all the nodes created by the given BQL file
stops. The command doesn't stop if it contains a source that generates infinite
tuples or is rewindable. Other non-rewindable sources such as ``file`` stopping
when it emits all tuples written in a file can work well with the command.

Sources generally need to be created with ``PAUSED`` keyword in the
:ref:`ref_stmts_create_source` statement. Without ``PAUSED``, a source can start
emitting tuples before all nodes in a topology can correctly be set up.
Therefore, a BQL file passed to the command should look like::

    CREATE PAUSED SOURCE source_1 TYPE ...;
    CREATE PAUSED SOURCE source_2 TYPE ...;
    ...
    CREATE PAUSED SOURCE source_n TYPE ...;

    ... CREATE STREAM, CREATE SINK, or other statements

    RESUME SOURCE source_1;
    RESUME SOURCE source_2;
    ...
    RESUME SOURCE source_n;

With the ``--save-uds`` option described later, it saves UDSs at the end of its
execution.

Basic Usage
-----------

::

    $ ./sensorbee runfile my_topology.bql

With options::

    $ ./sensorbee runfile -c sensorbee.yaml -s '' my_topology.bql

Configuration
-------------

``sensorbee runfile`` accepts the configuration file for ``sensorbee run``. It
only uses ``logging`` and ``storage`` sections. The configuration file may
contain other sections as well and the same file for ``sensorbee run`` can also
be used for ``sensorbee runfile``. See
:ref:`its configuration <ref_commands_sensorbee_run_config>` for details.

Flags and Options
-----------------

``--config path`` or ``-c path``

    This option receives the path of the configuration file. By default, the
    value is empty and no configuration file is used. This value can also be
    passed through ``SENSORBEE_CONFIG`` environment variable.

``--help`` or ``-h``

    When this flag is given, the command prints the usage of itself.

``--save-uds udss`` or ``-s udss``

    This option receives a list of names of UDSs separated by commas. UDSs
    listed in it will be saved at the end of execution. For example, when the
    option is ``-s "a,b,c"``, UDSs named ``a``, ``b``, and ``c`` will be saved.
    To save all UDSs in a topology, pass an empty string: ``-s ""``.

    By default, all UDSs will **not** be saved at the end of execution.

``--topology name`` or ``-t name``

    This option changes the name of the topology to be run with the given BQL
    file. The default name is taken from the file name of the BQL file. The name
    specified to this option will be used in log messages or saved UDS data.
    Especially, names of files containing saved UDS data has contains the name
    of the topology. Therefore, providing the same name as the topology that
    will be run by ``sensorbee run`` later on allows users to prepare UDSs
    including pre-trained machine learning models in advance.

.. _ref_commands_sensorbee_shell:

sensorbee shell or bql
======================

``sensorbee shell`` or ``bql`` starts a new shell to manipulate the SensorBee
server. The shell can be terminated by writing ``exit`` or typing ``C-d``.

Both ``sensorbee shell`` and ``bql`` have the same interface, but ``bql`` is
installed by default while the ``sensorbee`` command needs to be built manually
to run ``sensorbee shell``.

Basic Usage
-----------

To run ``sensorbee shell``,

::

    $ ./sensorbee shell -t my_topology
    my_topology>

To run ``bql``,

::

    $ bql -t my_topology
    my_topology>

Flags and options
-----------------

``--api-version version``

    This option changes the API version of the SensorBee server. The default
    value of this option is ``v1``.

``--help`` or ``-h``

    When this flag is given, the command prints the usage of itself.

``--topology name`` or ``-t name``

    The name of a topology to be manipulated can be specified through this
    option so that ``USE topology_name`` doesn't have to be used in the shell.
    The default value is an empty name, that is, no topology is specified.

``--uri``

    This option is used when the SensorBee server is running at non-localhost
    or using non-default port number (15601). The value should have a format
    like ``http://host:port/``. The default value of this option is
    ``http://localhost:15601/``.

.. _ref_commands_sensorbee_topology:

sensorbee topology
==================

``sensorbee topology``, or ``sensorbee t``, is used to manipulate topologies on
the SensorBee server.

.. note::

    This command is provided because the syntax of BQL statements that
    controls topologies has not been discussed enough yet.

The command consists of following subcommands:

``sensorbee topology create <name>`` or ``sensorbee t c <name>``

    This command creates a new topology on the SensorBee server. The ``<name>``
    argument is the name of the topology to be created. ``$?`` will be 0 if
    the command is successful. Otherwise, it'll be non-zero. The command fails
    if the topology already exists on the server.

``sensorbee topology drop <name>`` or ``sensorbee t drop <name>``

    This command drops an existing topology on the SensorBee server. The
    ``<name>`` argument is the name of the topology to be dropped. ``$?`` will
    be 0 if the command is successful. Otherwise, it'll be non-zero. The command
    doesn't fail even if the topology doesn't exist on the server.

``sensorbee topology list`` or ``sensorbee t l``

    This commands prints names of all topologies that the SensorBee server has,
    one name per line.

All commands share the same flags and options. Flags and options need to be
given after the subcommand name::

    $ ./sensorbee topology create --flag --option value my_topology

In this example, a flag ``--flag`` and an option ``--option value`` are
provided. The argument of the command, i.e. the name of topology, is
``my_topology``.

Flags and Options
-----------------

``--api-version version``

    This option changes the API version of the SensorBee server. The default
    value of this option is ``v1``.

``--help`` or ``-h``

    When this flag is given, the command prints the usage of itself.

``--uri``

    This option is used when the SensorBee server is running at non-localhost
    or using non-default port number (15601). The value should have a format
    like ``http://host:port/``. The default value of this option is
    ``http://localhost:15601/``.
