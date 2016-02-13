**********************
Using Machine Learning
**********************

This chapter describes how to use machine learning on SensorBee.

In this tutorial, SensorBee retrieves tweets from Twitter's public stream using
sample API. SensorBee adds two labels to each tweet: age and gender. Tweets are
labeled (*classified*) by machine learning.

Following sections shows how to install dependencies, set them up, and apply
machine learning to tweets using SensorBee.

.. note::

    Due to the way the Twitter client receives tweets from Twitter, the behavior
    of this tutorial demonstration doesn't seem very smooth. For example, it
    gets around 50 tweets in 100ms and stops for 900ms, then repeats the same
    behavior in every one second. So, it's easily misunderstood that SensorBee
    and its machine learning library are doing mini-batch processing, but they
    do not actually do it.

Prerequisities
==============

This tutorial requires following software to be installed:

* Ruby 2.1.8 or later

    * https://www.ruby-lang.org/en/documentation/installation/

* Elasticsearch 2.2.0 or later

    * https://www.elastic.co/products/elasticsearch

* Kibana 4.4.0 or later

    * https://www.elastic.co/products/kibana

In this tutorial, Ruby 2.1.4, Elasticsearch 2.2.0, and Kibana 4.4.0 are used.

This tutorial assumes that Elasticsearch and Kibana are running on the same
host as where SensorBee runs. However, it's can be configured to use those
servers running on a different host.

In addition, Go 1.4 or later is, of course, required. See
:ref:`tutorial_getting_started` to learn how to setting it up.

Quick Setting Up Guide
----------------------

If there's no Elasticsearch and Kibana instances that can be used for this
tutorial, they need to be installed. Skip this subsection if they're already
installed. In case an error occurs, look at documentation in
`<http://www.elastic.co/>`_.

Installing and Running Elasticsearch
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Download the package from `<https://www.elastic.co/downloads/elasticsearch>`_
and extract the compressed file. Then, run ``bin/elasticsearch`` in the
directory::

    /path/to/elasticserach-2.2.0$ bin/elasticsearch
    ... log messages ...

To see if Elasticsearch is running, access the server with ``curl`` command::

    $ curl http://localhost:9200/
    {
      "name" : "Peregrine",
      "cluster_name" : "elasticsearch",
      "version" : {
        "number" : "2.2.0",
        "build_hash" : "8ff36d139e16f8720f2947ef62c8167a888992fe",
        "build_timestamp" : "2016-01-27T13:32:39Z",
        "build_snapshot" : false,
        "lucene_version" : "5.4.1"
      },
      "tagline" : "You Know, for Search"
    }

Installing and Running Kibana
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Download the package from `<https://www.elastic.co/downloads/kibana>`_ and
extract the compressed file. Then, run ``bin/kibana`` in the directory::

    /path/to/kibana-4.4.0$ bin/kibana
    ... log messages ...

Access `<http://localhost:5601/>`_ with a Web browser. Kibana is running
correctly if it shows a page saying "Configure an index pattern". Since
Elasticsearch doesn't have the data yet, no more operation isn't necessary at
the moment. `Running SensorBee`_ section shows how to perform further steps
later.

Installation and Setup
======================

Some work needs to be done before actually trying to interact with this tutorial.

Installing the Tutorial Package
-------------------------------

To setting up the system, ``go get`` the tutorial package first::

    $ go get github.com/sensorbee/tutorial/ml
    $

The package contains configuration files that are necessary for the tutorial
under the ``config`` directory. Create a temporary directory and copy those
files to the directory (**replace /path/to/ with an appropriate path**)::

::

    $ mkdir -p /path/to/sbml
    $ cp $GOPATH/src/github.com/sensorbee/tutorial/ml/config/* /path/to/sbml/
    $ cd /path/to/sbml
    /path/to/sbml$ ls
    TODO

Installing and Running fluentd
------------------------------

This tutorial, and SensorBee, relies on `fluentd <http://www.fluentd.org/>`_.
fluentd is an open source data collector that provides many input and output
plugins to connect with a wide variety of databases including Elasticsearch.
Skip this subsection if fluentd is already installed.

To install fluentd for this tutorial, bundler needs to be installed with
the ``gem`` command. To see if it's already installed, run ``gem list``.
Something like ``bundler (1.11.2)`` shows up if it's already installed::

    /path/to/sbml$ gem list | grep bundler
    bundler (1.11.2)
    /path/to/sbml$

Otherwise, install bundler with ``gem install bundler``. It may require admin
privileges (i.e. ``sudo``)::

    /path/to/sbml$ gem install bundler
    Fetching: bundler-1.11.2.gem (100%)
    Successfully installed bundler-1.11.2
    Parsing documentation for bundler-1.11.2
    Installing ri documentation for bundler-1.11.2
    Done installing documentation for bundler after 3 seconds
    1 gem installed
    /path/to/sbml$

After installing bundler, run the following command to install fluentd and its
plugins under the ``/path/to/sbml`` directory::

    /path/to/sbml$ bundle install --path vendor/bundle
    Fetching gem metadata from https://rubygems.org/............
    Fetching version metadata from https://rubygems.org/..
    Resolving dependencies...
    Installing cool.io 1.4.3 with native extensions
    Installing multi_json 1.11.2
    Installing multipart-post 2.0.0
    Installing excon 0.45.4
    Installing http_parser.rb 0.6.0 with native extensions
    Installing json 1.8.3 with native extensions
    Installing msgpack 0.5.12 with native extensions
    Installing sigdump 0.2.4
    Installing string-scrub 0.0.5 with native extensions
    Installing thread_safe 0.3.5
    Installing yajl-ruby 1.2.1 with native extensions
    Using bundler 1.11.2
    Installing elasticsearch-api 1.0.15
    Installing faraday 0.9.2
    Installing tzinfo 1.2.2
    Installing elasticsearch-transport 1.0.15
    Installing tzinfo-data 1.2016.1
    Installing elasticsearch 1.0.15
    Installing fluentd 0.12.20
    Installing fluent-plugin-elasticsearch 1.3.0
    Bundle complete! 2 Gemfile dependencies, 20 gems now installed.
    Bundled gems are installed into ./vendor/bundle.
    /path/to/sbml$

With ``--path vendor/bundle`` option, all Ruby gems required for this tutorial
is locally installed in the ``/path/to/sbml/vendor/bundle`` directory. To
confirm whether fluentd is correctly installed, run the command below::

    /path/to/sbml$ bundle exec fluentd --version
    fluentd 0.12.20
    /path/to/sbml$

If it prints the version, the installation is completed and fluentd is ready to
be used.

Once fluentd is installed, run it with the provided configuration file::

    /path/to/sbml$ bundle exec fluentd -c fluent.conf
    2016-02-05 16:02:10 -0800 [info]: reading config file path="fluent.conf"
    2016-02-05 16:02:10 -0800 [info]: starting fluentd-0.12.20
    2016-02-05 16:02:10 -0800 [info]: gem 'fluentd' version '0.12.20'
    2016-02-05 16:02:10 -0800 [info]: gem 'fluent-plugin-elasticsearch' version '1.3.0'
    2016-02-05 16:02:10 -0800 [info]: adding match pattern="sensorbee.tweets" type="...
    2016-02-05 16:02:10 -0800 [info]: adding source type="forward"
    2016-02-05 16:02:10 -0800 [info]: using configuration file: <ROOT>
      <source>
        @type forward
        @id forward_input
      </source>
      <match sensorbee.tweets>
        @type elasticsearch
        host localhost
        port 9200
        include_tag_key true
        tag_key @log_name
        logstash_format true
        flush_interval 1s
      </match>
    </ROOT>
    2016-02-05 16:02:10 -0800 [info]: listening fluent socket on 0.0.0.0:24224
    ^C2016-02-05 16:02:18 -0800 [info]: shutting down fluentd
    2016-02-05 16:02:18 -0800 [info]: shutting down input type="forward" plugin_id="...
    2016-02-05 16:02:18 -0800 [info]: shutting down output type="elasticsearch" plug...
    2016-02-05 16:02:18 -0800 [info]: process finished code=0

Some log messages are truncated with ``...`` at the end of each line.

The configuration file ``fluent.conf`` is provided as a part of this tutorial.
It defines one data source using ``in_forward`` and one destination that
is connected to Elasticsearch. If the Elasticserver is running on a different
host or using a port number different from 9200, edit ``fluent.conf``::

    <source>
      @type forward
      @id forward_input
    </source>
    <match sensorbee.tweets>
      @type elasticsearch
      host {custom host name}
      port {custom port number}
      include_tag_key true
      tag_key @log_name
      logstash_format true
      flush_interval 1s
    </match>

Also, feel free to change other parameters to adjust the configuration to the
actual environment. Parameters for the Elasticsearch plugin are described at
`<https://github.com/uken/fluent-plugin-elasticsearch>`_.

Create Twitter API Key
----------------------

TODO: Visit Twitter
TODO: Create api_key.yaml

Running SensorBee
=================

All requirements for this tutorial have been installed and set up. The next
step is to build and run ``sensorbee`` command::

    /path/to/sbml$ build_sensorbee
    sensorbee_main.go
    /path/to/sbml$ ./sensorbee run -c sensorbee.yaml
    INFO[0000] Setting up the server context
    INFO[0000] Setting up the topology                       topology=twitter
    INFO[0000] Starting the server on :15601

Because SensorBee loads pre-trained machine learning models on its startup,
it may take a while to setting up a topology. After the server shows the
message ``Starting the server on :15601``, access Kibana at
`<http://localhost:5601/>`_. If operations so far are sucessful, it returns the
page as shown below:

.. image:: /tutorial/kibana_create_index.png

Click "Create" button to work with data coming from SensorBee. After the action
is completed, Kibana is ready to visualize data. The picture below shows an
example chart:

.. image:: /tutorial/kibana_chart_sample.png

Although this tutorial doesn't describe the usage of Kibana, many tutorials
and examples can be found on the Web.

Troubleshooting
---------------

If Kibana doesn't show the "Create" button, something may not be working
properly. First, enter ``sensorbee shell`` to see SensorBee is working::

    /path/to/sbml$ sensorbee shell -t twitter
    twitter>

Then, issue the following ``SELECT`` statement::

    twitter> SELECT RSTREAM * FROM public_tweets [RANGE 1 TUPLES];
    ... tweets show up here ...

If the statement returns an error or it doesn't show any tweet:

1. the host may not be connected to Twitter. Check the internet connection with
   commands such as ``ping``.
2. The API key written in ``api_key.yaml`` may be wrong.

When the statement above shows tweets, query another stream::

    twitter> SELECT RSTREAM * FROM labeled_tweets [RANGE 1 TUPLES];
    ... tweets show up here ...

If the statement doesn't show any tweets, the format of tweets may have been
changed since the time of this writing. If so, modify BQL statements in
``twitter.bql`` to support the new format. `How BQL Works`_ describes what
each statement does.

When the statement above prints tweets, fluentd or Elasticsearch may have not
been staretd yet. Check they're running correctly.

For other errors, report them to `<https://github.com/sensorbee/tutorial>`_.

How BQL Works
=============

TODO: explain some streams defined in BQL file
TODO: get some tuples from some streams

Open another terminal to launch ``sensorbee shell``::

    /path/to/sbml$ ./sensorbee shell -t twitter
    (twitter)>>>

Training
========

::

    /path/to/sbml$ ./sensorbee runfile -t twitter -c sensorbee.yaml -s '' train.bql

Evaluation
----------

Evaluation tools are being developed.