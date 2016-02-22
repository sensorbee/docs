**********************
Using Machine Learning
**********************

This chapter describes how to use machine learning on SensorBee.

In this tutorial, SensorBee retrieves tweets written in English from
Twitter's public stream using sample API. SensorBee adds two labels to each
tweet: age and gender. Tweets are labeled (*classified*) by machine learning.

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

This tutorial requires Twitter's API keys. To create keys, visit
`Application Management <https://apps.twitter.com/>`_. Once a new application is
created, click the application and its "Keys and Access Tokens" tab. The page
should show 4 keys:

* Consumer Key (API Key)
* Consumer Secret (API Secret)
* Access Token
* Access Token Secret

Then, create the ``api_key.yaml`` in the ``/path/to/sbml`` directory and copy
keys to the file as follows::

    /path/to/sbml$ cat api_key.yaml
    consumer_key: <Consumer Key (API Key)>
    consumer_secret: <Consumer Secret (API Secret)>
    access_token: <Access Token>
    access_token_secret: <Access Token Secret>

Replace each key's value with the actual values shown in Twitter's application
management page.

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
``twitter.bql`` to support the new format. `BQL Statements and Plugins`_
describes what each statement does.

When the statement above prints tweets, fluentd or Elasticsearch may have not
been staretd yet. Check they're running correctly.

For other errors, report them to `<https://github.com/sensorbee/tutorial>`_.

BQL Statements and Plugins
==========================

This section describes how SensorBee input tweets from Twitter, preprocesses
tweets for machine learning, and finally classifies tweets to extract
demographic information of each tweets. ``twitter.bql`` in the ``config``
directory contains all BQL statements used in this tutorial.

Following subsections explains what each statement does. To interact with some
streams created by ``twitter.bql``,  open another terminal to launch
``sensorbee shell``::

    /path/to/sbml$ ./sensorbee shell -t twitter
    twitter>

Creating a Twitter Source
-------------------------

This tutorial doesn't work without retrieving the public timeline of Twitter
using the Sample API. The Sample API is provided for free to retrieve a
portion of tweets sampled from the public timeline.

`github.com/sensorbee/twitter <https://github.com/sensorbee/twitter/>`_
package provides a plugin for public time line retrieval. Its type name is
``twitter_public_stream``. The plugin can be registered to the SensorBee
server by adding ``github.com/sensorbee/tiwtter/plugin`` to the ``build.yaml``
configuration file for ``build_sensorbee``.

::

    CREATE SOURCE public_tweets TYPE twitter_public_stream
        WITH key_file = 'api_key.yaml';

This statement creates a new source ``public_tweets``. To retrieve raw tweets
from the source run the following ``SELECT`` statement::

    twitter> SELECT RSTREAM * FROM public_tweets [RANGE 1 TUPLES];

.. note::

    For simplicity, a relative path is specified for ``key_file`` parameter.
    However, it's usually recommended to pass an absolute path for it when
    running the SensorBee server as a daemon.

Preprocessing Tweets and Extracting Features for Machine Learning
-----------------------------------------------------------------

Before applying machine learning to tweets, they need to be converted into
another form of information so that machine learning algorithms can utlize
them. The conversion consists of two tasks: preprocessing and feature
extraction. Preprocessing generally involves data cleansing, filtering,
normalization, and so on. Feature extraction transforms preprocessed data
into several pieces of information (i.e. features) that machine learning
algorithms can "understand".

Which preprocessing or feature extraction methods are required for machine
learning varies depending on the format or data type of input data or machine
learning algorithms to be used. Therefore, this tutorial only shows one
example of applying a classification algorithm to English tweets.

Selecting Meaningful Fields of English Tweets
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Because this tutorial aims at English tweets, tweets written in other
languages needs to be removed. This can be done by the ``WHERE``
clause::

    SELECT RSTREAM * FROM public_tweets [RANGE 1 TUPLES]
        WHERE lang = 'en';

Tweets have the ``lang`` field and it can be used for the filtering.

In addition to it, not all fields in a raw tweet will be required for machine
learning. Thus, removing unnecessary fields keeps data simple and clean::

    CREATE STREAM en_tweets AS
        SELECT RSTREAM
            'sensorbee.tweets' AS tag, id_str AS id, lang, text,
            user.screen_name AS screen_name, user.description AS description
        FROM public_tweets [RANGE 1 TUPLES]
        WHERE lang = 'en';

This statement creates a new stream ``en_tweets``. The resulting from the
stream will look like::

    {
        'tag': 'sensorbee.tweets',
        'id': 'the string representation of tweet's id',
        'lang': 'en',
        'text': 'the contents of the tweet',
        'screen_name': 'user's @screen_name',
        'description': 'user's profile description'
    }

.. note::

    ``AS`` in ``user.screen_name AS screen_name`` is required at the moment.
    Without it, the field would have the name like ``col_n``. This is because
    ``user.screen_name`` could be evaluated as a JSON Path and might result in
    multiple return values so that it cannot properly be named. This
    specification might be going to be changed in the future version.

Removing Noise
^^^^^^^^^^^^^^

A noise that is meaningless and could be harmful to machine learning
algorithms needs to be removed. The field of natural language processing
(NLP) have developed many methods for this purpose and they can be found in a
wide variety of articles. However, this tutorial only applies some of the
most basic operations on each tweets.

::

    CREATE STREAM preprocessed_tweets AS
        SELECT RSTREAM
            filter_stop_words(
                nlp_split(
                    nlp_to_lower(filter_punctuation_marks(text)),
                ' ')) AS text_vector,
            filter_stop_words(
                nlp_split(
                    nlp_to_lower(filter_punctuation_marks(description)),
                ' ')) AS description_vector,
            *
        FROM en_tweets [RANGE 1 TUPLES];

The statement above creates a new stream ``preprocessed_tweets`` from
``en_tweets``. It adds two fields to the tuple emitted from ``en_tweets``:
``text_vector`` and ``description_vector``. As for preprocessing, the
statement applies following methods to ``text`` and ``description`` fields:

* Removing punctuation marks
* Changing uppercase letters to lowercase
* Removing stopwords

.. todo:: rename "stop word" to "stopword" in both code and BQL

First of all, punctuation marks are removed by the user-defined function (UDF)
``filter_puncuation_marks``. It's provided as a plugin of this tutorial in
``github.com/sensorbee/tutorial/ml`` package. The UDF removes some punctuation
marks such as ",", ".", or "()".

.. note::

    Emoticons such as ":)" may play a very important role in classification
    tasks like sentiment estimation. However, ``filter_punctuation_marks``
    simply removes most of them for simplicity. Develop a better UDF to solve
    this issue as an exercise.

Second of all, all uppercase letters are converted into lowercase letters by
the ``nlp_to_lower`` UDF. The UDF is registered in
``github.com/sensorbee/nlp/plugin``. Because a letter is mere byte code and
the values of 'a' and 'A' are different, machine learning algorithms consider
"word" and "Word" have different meanings. To avoid that confusion, all letter
should be "normalized".

.. note::

    Of course, some words should be distinguished by explicitly starting with
    an uppercase. For example, "Mike" could be a name of a person, but
    changing it to "mike" could make the word vague.

Finally, all stopwords are removed. Stopwords are words that appear too often
and don't provide any insight for classification. Stopword filtering in this
tutorial is done in two steps: tokenization and filtering. To perform a
dictionary-based stopword filtering, the content of a tweet need to be
tokenized. Tokenization is a process that converts a sentence into a sequence
of words. In English, "I like sushi" will be tokenized as
``['I', 'like', 'sushi']``. Although tokenization isn't as simple as just
splitting words by white spaces, the ``preprocessed_tweets`` stream simply
does it for simplicity by the UDF ``nlp_split``, which is defined in
``github.com/sensorbee/nlp`` package. ``nlp_split`` takes two arguments: a
sentence and a splitter. In the statement, contents are split by a white
space. ``nlp_split`` returns an array of strings. Then, the UDF
``filter_stop_words`` takes the return value of ``nlp_split`` and remove
stopword contained in the array. ``filter_stop_word`` is provided as a part
of this tutorial in ``github.com/sensorbee/tutorial/ml`` package. It's a mere
example UDF and doesn't provide perfect stopword filtering.

As a result, both ``text_vector`` and ``description_vector`` have an array
of words like ``['i', 'want', 'eat', 'sushi']`` created from the sentence
``I want to eat sushi.``.

Preprocessing shown so far is very similar to the preprocessing required for
full-text search engines. There should be many valuable resources among that
field including Elasticsearch.

.. note::

    For other preprocessing approaches such as stemming, refer natural
    language processing textbooks.

Creating Features
^^^^^^^^^^^^^^^^^

In NLP, a bag-of-words representation is usually used as a feature for
machine learning algorithms. A bag-of-words consists of pairs of a word and
its weight. Weight could be any numerical value and usually something related
to term frequency (TF) is used. A sequence of the pairs is called a feature
vector.

A feature vector can be expressed as an array of weights. Each word in all
tweets observed by a machine learning algorithm corresponds to a particular
position of the array. For example, the weight of the word "want" may be 4th
element of the array.

A feature vector for NLP data could be very long because tweets contains many
words. However, each vector would be sparse due to the maximum length of
tweets. Even if machine learning algorithms observe more than 100,000 words
and use them as features, each tweet only contains around 30 or 40 words.
Therefore, each feature vector is very sparse, that is, only a small number
its elements have non-zero weight. In such cases, a feature vector can
effectively expressed as a map::

    {
        'word': weight,
        'word': weight,
        ...
    }

This tutorial uses online classification algorithms that is imported from
Jubatus. It accepts the following form of data as a feature vector::

    {
        'word1': 1,
        'key1': {
            'word2': 2,
            'word3': 1.5,
        },
        'word4': [1.1, 1.2, 1.3]
    }

A map can be nested and its value can be an array containing weights. The map
above is converted to something like::

    {
        'word1': 1,
        'key1/word2': 2,
        'key1/word3': 1.5,
        'word4[0]': 1.1,
        'word4[1]': 1.2,
        'word4[2]': 1.3
    }

The actual feature vector for the tutorial is created by the ``fv_tweets``
stream::

    CREATE STREAM fv_tweets AS
    SELECT RSTREAM
        {
            'text': nlp_weight_tf(text_vector),
            'description': nlp_weight_tf(description_vector)
        } AS feature_vector,
        tag, id, screen_name, lang, text, description
    FROM preprocessed_tweets [RANGE 1 TUPLES];

As described earler, ``text_vector`` and ``description_vector`` are arrays of
words. ``nlp_weight_tf`` function defined in ``github.com/sensorbee/nlp``
package computes a feature vector from the array. The weight is term
frequency (i.e. the number of occurrances of a word). The result is a map
expressing a sparse vector above. To see how the ``feature_vector`` looks
like, just issue a ``SELECT`` statement for the ``fv_tweets`` stream.

All required preprocessing and feature extraction have been completed and
it's now ready to apply machine learning to tweets.

Applying Machine Learning
-------------------------

The ``fv_tweets`` stream now has all the information required by a machine
learning algorithm to classify tweets. To apply the algorithm for each tweets,
pre-trained machine learning models have to be loaded::

    LOAD STATE gender_model TYPE jubaclassifier_arow
        OR CREATE IF NOT SAVED
        WITH label_field = 'gender', regularization_weight = 0.001;
    LOAD STATE age_model TYPE jubaclassifier_arow
        OR CREATE IF NOT SAVED
        WITH label_field = 'age', regularization_weight = 0.001;

In SensorBee, Machine learning models are expressed as user-defined states
(UDSs). In the statement above, two models are loaded: ``gender_model`` and
``age_model``. These models has necessary information to classify gender and
age of the user of each tweet. They're saved in the ``uds`` directory
beforehand::

    /path/to/sbml$ ls uds
    twitter-age_model-default.state
    twitter-gender_model-default.state

These filenames were automatically assigned by SensorBee server when the
``SAVE STATE`` statement is issued. It'll be described later.

Both models have the type ``jubaclassifier_arow`` imported from
Jubatus, which distributed online machine learning server. The UDS type is
implemented in the `github.com/sensorbee/jubatus/classifier <https://github.com/sensorbee/jubatus/classifier>`_
package. ``jubaclassifier_arow`` implements AROW online linear classification
algorithm [Crammer09]_. Parameters specified in the ``WITH`` clause are related
to training and will be described later.

After loading the models as UDSs, the machine learning algorithm is ready
to work::

    CREATE STREAM labeled_tweets AS
        SELECT RSTREAM
            juba_classified_label(jubaclassify('gender_model', feature_vector)) AS gender,
            juba_classified_label(jubaclassify('age_model', feature_vector)) AS age,
            tag, id, screen_name, lang, text, description
        FROM fv_tweets [RANGE 1 TUPLES];

The ``labeled_tweets`` stream emits tweets with ``gender`` and ``age`` labels.
The ``jubaclassify`` UDF performs classification based on the given model.

::

    twitter> EVAL jubaclassify("gender_model", {
        "text": {"i": 1, "wanna": 1, "eat":1, "sushi":1},
        "description": {"i": 1, "need": 1, "sushi": 1}
    });
    {"male":0.021088751032948494,"female":-0.020287269726395607}

``jubaclassify`` returns a map of labels and their scores as shown above. The
higher the score of a label, the more likely a tweet has the label. To choose
the label having the highest score, the ``juba_classified_label`` function is
used::

    twitter> EVAL juba_classified_label({
        "male":0.021088751032948494,"female":-0.020287269726395607});
    "male"

``jubaclassify`` and ``juba_classified_label`` functions are also defined in
the ``github.com/sensorbee/jubatus/classifier`` package.

.. [Crammer09] Koby Crammer, Alex Kulesza and Mark Dredze, Adaptive Regularization Of Weight Vectors, Advances in Neural Information Processing Systems, 2009

Inserting Labeled Tweets Into Elasticsearch via Fluentd
-------------------------------------------------------

Finally, tweets labeled by machine learning need to be inserted into
Elasticsearch for visualization. This is done via fluentd which is previously
set up.

::

    CREATE SINK fluentd TYPE fluentd;
    INSERT INTO fluentd from labeled_tweets;

SensorBee provides ``fluentd`` plugins in the ``github.com/sensorbee/fluentd``
package. The ``fluentd`` sink write tuples into fluentd's ``forward`` input
plugin running on the same host.

After creating the sink, the ``INSERT INTO`` statement starts writing tuples
from a source or a stream into it.

Training
========

::

    /path/to/sbml$ ./sensorbee runfile -t twitter -c sensorbee.yaml -s '' train.bql

Evaluation
----------

Evaluation tools are being developed.
