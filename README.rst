Nameko Redis Keyspace Notifications
===================================

.. pull-quote::

    Nameko_ `Redis Keyspace Notifications`_ extension.


.. image:: https://img.shields.io/pypi/v/nameko-rediskn.svg
    :target: https://pypi.org/project/nameko-rediskn/

.. image:: https://img.shields.io/pypi/pyversions/nameko-rediskn.svg
    :target: https://pypi.org/project/nameko-rediskn/

.. image:: https://img.shields.io/pypi/format/nameko-rediskn.svg
    :target: https://pypi.org/project/nameko-rediskn/

.. image:: https://travis-ci.org/sohonetlabs/nameko-rediskn.svg?branch=master
    :target: https://travis-ci.org/sohonetlabs/nameko-rediskn


Usage
-----

This Nameko_ extension adds the ability to subscribe to events, keys and
databases.

Some event examples:

- ``expire`` events fired for ``EXPIRE`` commands
- ``expired`` events fired when a key gets deleted due to expiration

Usage example:

 .. code-block:: python

    from nameko_rediskn import rediskn, REDIS_PMESSAGE_TYPE


    class MyService:

        name = 'my-service'

        @rediskn.subscribe(uri_config_key='MY_REDIS', keys='foo/bar-*')
        def subscriber(self, message):
            if message['type'] != REDIS_PMESSAGE_TYPE:
                return

            event_type = message['data']
            if event_type != 'expired':
                return

            key = message['channel'].split(':')[1]

            # ...

Where ``subscribe`` accepts:

- ``MY_REDIS``, which is the attribute name referring to he Redis URI
  (see the Configuration_ section below).
- ``events``, ``keys`` and ``dbs`` as a single value (string) or a
  list of values to subscribe to. They are all optional but at least one
  of those arguments must be provided.

For more information, you can check the documentation of the
``RedisKNEntrypoint`` entrypoint.

**NOTE**: this dependency is not "cluster-aware" and fires on all service
instances. There are different ways to solve that: using ddebounce_ is
one of them.


Configuration
-------------

Nameko_ configuration file:

 .. code-block:: yaml

    # config.yaml

    REDIS:
        notification_events: "KEA"

    REDIS_URIS:
        MY_REDIS: "redis://localhost:6380/0"

``REDIS[notification_events]`` is optional and can be omited or just
contain ``None``. Otherwise, it must have a valid value for the
``'notify-keyspace-events'`` Redis client configuration attribute. This
should be ideally set on the server side, as setting it in one of the
Redis clients will affect the rest of them.

``REDIS_URIS`` follows the config format used by the `Nameko Redis`_
dependency provider, where ``MY_REDIS`` is just the attribute name
refering to the Redis URI of the instance being used.


Tests
-----

It is assumed that **RabbitMQ** is up and running on the default URI
``guest:guest@localhost`` and uses the default ports.

**Redis** should be also running on the default port.

There are Makefile targets to run both RabbitMQ and Redis docker
containers locally using the default ports and configuration:

 .. code-block:: shell

    $ make rabbitmq-container
    $ make redis-container

To run the tests locally:

.. code-block:: shell

    $ # Create/activate a virtual environment
    $ pip install tox
    $ tox

There are other Makefile targets to run the tests, but the extra
dependencies will have to be installed:

.. code-block:: shell

    $ pip install -U --editable ".[dev]"
    $ make test
    $ make coverage

A different RabbitMQ URI can be provided overriding the following
environment variables: ``RABBIT_CTL_URI`` and ``AMQP_URI``.

Additional ``pytest`` parameters can be also provided using the ``ARGS``
variable:

.. code-block:: shell

    $ make test RABBIT_CTL_URI=http://guest:guest@localhost:15673 AMQP_URI=amqp://guest:guest@localhost:5673 ARGS='-x -vv --disable-warnings'
    $ make coverage RABBIT_CTL_URI=http://guest:guest@localhost:15673 AMQP_URI=amqp://guest:guest@localhost:5673 ARGS='-x -vv --disable-warnings'


Nameko support
--------------

The following Nameko_ versions are actively supported: ``2.11``,
``2.12``.

However, this extension should work from, at least, Nameko_ ``2.6``
onwards.


Redis support
-------------

The following `Redis Python`_ versions are actively supported: ``2.10``,
``3.0``, ``3.1``, ``3.2``.

Redis_ ``4.0`` is actively supported.


Changelog
---------

Consult the CHANGELOG_ document for fixes and enhancements of each
version.


License
-------

The MIT License. See LICENSE_ for details.


.. _Nameko: http://nameko.readthedocs.org
.. _Redis Python: https://github.com/andymccurdy/redis-py
.. _Redis: https://redis.io
.. _Redis Keyspace Notifications: https://redis.io/topics/notifications
.. _Nameko Redis: https://github.com/etataurov/nameko-redis
.. _CHANGELOG: https://github.com/sohonetlabs/nameko-rediskn/blob/master/CHANGELOG.rst
.. _LICENSE: https://github.com/sohonetlabs/nameko-rediskn/blob/master/LICENSE
.. _ddebounce: https://github.com/iky/ddebounce
