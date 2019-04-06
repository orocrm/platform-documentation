.. _op-structure--mq--rabbitmq :

RabbitMQ Transport (via AmqpOroMessageQueue Bundle)
===================================================

The bundle registers AmqpTransportFactory.

AMQP (RabbitMQ) Transport
-------------------------

RabbitMQ provides better and faster messages delivery as opposed to DBAL.
It is recommended to use RabbitMQ, if possible.

Options
~~~~~~~

The config settings for the `default RabbitMQ Access Control
settings <https://www.rabbitmq.com/access-control.html>`__ (a user named
guest with the default password of guest, granted full access to the /
virtual host) are the following:

.. code:: yaml

    # config/config.yml

    oro_message_queue:
      transport:
        default: 'amqp'
        amqp:
            host: 'localhost'
            port: '5672'
            user: 'guest'
            password: 'guest'
            vhost: '/'

We can also move the specified options to the ``parameters.yml``:

.. code:: yaml

    # config/config.yml

    oro_message_queue:
        transport:
            default: '%message_queue_transport%'
            '%message_queue_transport%': '%message_queue_transport_config%'
        client: ~

.. code:: yaml

    # config/parameters.yml

        message_queue_transport: 'amqp'
        message_queue_transport_config: { host: 'localhost', port: '5672', user: 'guest', password: 'guest', vhost: '/' }

RabbitMQ installation
---------------------

You need to have RabbitMQ **version 3.6.**\ \* installed to use the AMQP
transport. To install the RabbitMQ you should follow the `download and
installation manual <https://www.rabbitmq.com/download.html>`__.

After the installation, please check that you have all the required plugins
installed and enabled.

RabbitMQ plugins
----------------

Required plugins
~~~~~~~~~~~~~~~~

+---------------+-------------+---------------+
| Plugin name   | Version     | Appointment   |
+===============+=============+===============+
| rabbitmq\_del | 20171215    | A plugin that |
| ayed\_message |             | adds          |
| \_exchange    |             | delayed-messa |
|               |             | ging          |
|               |             | (or           |
|               |             | scheduled-mes |
|               |             | saging)       |
|               |             | to RabbitMQ.  |
|               |             | `See          |
|               |             | also <https:/ |
|               |             | /github.com/r |
|               |             | abbitmq/rabbi |
|               |             | tmq-delayed-m |
|               |             | essage-exchan |
|               |             | ge>`__        |
+---------------+-------------+---------------+

The plugin ``rabbitmq_delayed_message_exchange`` is necessary
for the proper work but it is not installed by default, so you need to
download, install and enable it.

To download it, use the following command:

.. code-block:: none
    :linenos:

    wget https://dl.bintray.com/rabbitmq/community-plugins/3.6.x/rabbitmq_delayed_message_exchange/rabbitmq_delayed_message_exchange-20171215-3.6.x.zip && unzip rabbitmq_delayed_message_exchange-20171215-3.6.x.zip -d {RABBITMQ_HOME}/plugins && rm rabbitmq_delayed_message_exchange-20171215-3.6.x.zip

To enable it, use the following command:

.. code-block:: none
    :linenos:

    rabbitmq-plugins enable --offline rabbitmq_delayed_message_exchange

Recommended plugins
~~~~~~~~~~~~~~~~~~~

+----------------------+-------------+---------------+
| Plugin name          | Version     | Appointment   |
+======================+=============+===============+
| rabbitmq\_management | 3.6.*       |Provides       |
|                      |             | an            |
|                      |             | HTTP-based    |
|                      |             | API for       |
|                      |             | management    |
|                      |             | and           |
|                      |             | monitoring    |
|                      |             | of your       |
|                      |             | RabbitMQ      |
|                      |             | server.       |
|                      |             | `See          |
|                      |             | also <https   |
|                      |             | ://www.rabb   |
|                      |             | itmq.com/ma   |
|                      |             | nagement.ht   |
|                      |             | ml>`__        |
+----------------------+-------------+---------------+

Plugins management
~~~~~~~~~~~~~~~~~~

To enable plugins, use the ``rabbitmq-plugins`` tool:
``rabbitmq-plugins enable plugin-name``

And to disable plugins again, use:
``rabbitmq-plugins disable plugin-name``

To see the list of enabled plugins, use:
``rabbitmq-plugins list  -e``

You will see something like:

.. code-block:: none
    :linenos:

    [e*] amqp_client                       3.6.5
    [e*] mochiweb                          2.13.1
    [E*] rabbitmq_delayed_message_exchange 20171215
    [E*] rabbitmq_management               3.6.5
    [e*] rabbitmq_management_agent         3.6.5
    [e*] rabbitmq_web_dispatch             3.6.5
    [e*] webmachine                        1.10.3

The sign ``[E*]`` means that the plugin was explicitly enabled, i.e.
somebody enabled it manually. The sign ``[e*]`` means the plugin was
implicitly enabled, i.e. enabled automatically as it was required for
a different enabled plugin.

`More about RabbitMQ plugins <https://www.rabbitmq.com/community-plugins.html>`__

`More about RabbitMQ plugins management <https://www.rabbitmq.com/plugins.html>`__

Troubleshooting
---------------

The following exception

.. code-block:: none
    :linenos:

    [PhpAmqpLib\Exception\AMQPRuntimeException]
    Broken pipe or closed connection

might be caused by one of the following reasons:

-  The plugin ``rabbitmq_delayed_message_exchange`` is missing.
-  The RabbitMQ version is too old (older than 3.5.8).

RabbitMQ Useful Hints
---------------------

-  You can see the RabbitMQ default web interface here, if the
   ``rabbitmq_management`` plugin is enabled:
   ``http://localhost:15672/``. `See more details
   here <https://www.rabbitmq.com/management.html>`__.
-  You can temporary stop RabbitMQ by running the command
   ``rabbitmqctl stop_app``. The command will stop the RabbitMQ
   application, leaving the Erlang node running. You can resume it with
   the command ``rabbitmqctl start_app``. `See more details
   here <https://www.rabbitmq.com/man/rabbitmqctl.1.man.html>`__.
