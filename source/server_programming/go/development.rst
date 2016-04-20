.. _server_programming_go_development_flow:

Development Flow of Components in Go
====================================

The typical development flow of components like a UDF, a UDS type, a source type,
or a sink type should be discussed before looking into details of each component.

The basic flow is as follows:

#. Create a git repository for components
#. Implement components
#. Create a plugin subpackage in the repository

Create a Git Repository for Components
--------------------------------------

Components are written in Go, so they need to be in a valid git repository (or
a repository of a different version control system). One repository may provide
multiple types of components. For example, a repository could have 10 UDFs, and
5 UDS types, 2 source types, and 1 sink type. However, since Go is very well designed to
provide packages in a fine-grained manner, each repository should only provide
a minimum set of components that are logically related and make sense to be in
the same repository.

Implement Components
--------------------

The next step is to implement components. There is no restriction on which
standard or 3rd party packages to depend on.

Functions or structs that are to be registered to the SensorBee server need to be
referred to by the plugin subpackage, which is described in the next subsection.
Thus, names of those symbols need to start with a capital letter.

In this step, components should not be registered to the SensorBee server yet.

Create a Plugin Subpackage in the Repository
--------------------------------------------

It is highly recommended that the repository has a separate package (i.e. a
subdirectory) which only registers components to the SensorBee server. There is
usually one file named "plugin.go" in the plugin package and it only contains a
series of registration function calls in ``init`` function. For instance, if the
repository only provides one UDF, the contents of ``plugin.go`` would look
like:

.. code-block:: go

    // in github.com/user/myudf/plugin/plugin.go
    package plugin

    import (
        "gopkg.in/sensorbee/sensorbee.v0/bql/udf"
        "github.com/user/myudf"
    )

    func init() {
        udf.MustRegisterGlobalUDF("my_udf", &myudf.MyUDF{})
    }

There are two reasons to have a plugin subpackage separated from the
implementation of components. First, by separating them, other Go packages can
import the components to use the package as a library without registering them
to SensorBee. Second, having a separated plugin package allows a user to
register a component with a different name. This is especially useful
when names of components conflict each other.

To use the example plugin above, the ``github.com/user/myudf/plugin`` package needs
to be added to the plugin path list of SensorBee.

Repository Organization
-----------------------

The typical organization of the repository is

* ``github.com/user/repo``

    * ``README``: description and the usage of components in the repository
    * ``.go`` files: implementation of components
    * ``plugin/``: a subpackage for the plugin registration

        * ``plugin.go``

    * ``othersubpackages/``: there can be optional subpackages
