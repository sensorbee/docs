##################
Server Programming
##################

This part describes the extensibility of the SensorBee server. Topics covered
in this part are advanced and should be read after understanding basics of
SensorBee and BQL.

Because SensorBee is mainly written in
`The Go Programming Language <https://golang.org/>`_, understanding the language
before reading this part is also recommended.
`A Tour of Go <https://tour.golang.org/>`_ is an official tutorial and is one
of the best tutorial of the language. It runs on web browsers and doesn't
require any additional software installation. After learning the language,
`How to Write Go Code <https://golang.org/doc/code.html>`_ helps understand
how to use the go tool and the standard way to develop Go packages and
applications.

This part assumes that the go tool is installed and the developing environment
including Go's environment variables like ``GOROOT`` or ``GOPATH`` is
appropriately set up. SensorBee requires Go 1.4 or later.

.. include:: /server_programming/extend.rst
.. include:: /server_programming/go.rst
.. include:: /server_programming/python.rst
