.. _server_programming_go_development_flow:

Development Flow of Components in Go
====================================

The typical development flow of components like a UDF, a UDS, a source, or a
sink should be discussed before looking into details of each component.

The basic flow is as follows:

#. Create a git repository for components
#. Implement components
#. Create a plugin subpackage in the repository

Creating a Git Repository for Components
----------------------------------------

Components are written in Go, so they needs to be in a valid git repository (or
a repository of other version control systems). One repository may provide
multiple types of components. For example, a repository could have 10 UDFs, and
5 UDSs, 2 sources, and 1 sink. However, since Go is very well designed to
provide packages in a fine-grained manner, each repository should only provide
a minimum set of components that are logically related and make sense to be in
the same repository.

Implementing Components
-----------------------

The next step is to implement components. There's no restiction on which
standard or 3rd party packages to depend.

Functions or structs that are registered to the SensorBee server needs to be
referred by the plugin subpackage, which is described in the next subsection.
Thus, names of those symbols need to start with a capital letter.

In this step, components shouldn't be registered to the SensorBee server yet.

Creating a Plugin Subpackage in the Repository
----------------------------------------------

It is highly recommended that the repository have a separate package (i.e. a
subdirectory) which only registers components to the SensorBee server. There's
usually one file named "plugin.go" in the plugin package and it only contains a
series of registration function calls in ``init`` function. For instance, if the
repository only provides one UDF, the contents of "plugin.go" would be something
like::

    // in github.com/user/myudf/plugin/plugin.go
    package plugin

    import (
        "gopkg.in/sensorbee/sensorbee.v0/bql/udf"
        "github.com/user/myudf"
    )

    func init() {
        udf.MustRegisterGlobalUDF("my_udf", &myudf.MyUDF{})
    }

There're two reasons to have a plugin subpackage separated from the
implementation of components. Firstly, by separating them, other Go packages can
import the components to use the package as a library without registering them
to SensorBee. Secondly, having a separated plugin package allows a user to
register a component with a different name. This is especially useful
when names of comopnents conflict each other.

To use the example plugin above, "github.com/user/myudf/plugin" package needs
to be added to the plugin path list of SensorBee.

Repository Organization
-----------------------

The typical organization of the repository is

* github.com/user/repo

    * README: description and the usage of components in the repository
    * .go files: implementation of components
    * plugin/: a subpackage for the plugin registration

        * plugin.go

    * othersubpackages/: there can be optional subpackages
