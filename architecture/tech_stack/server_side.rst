.. _architecture-overview--tech-stack--server:

.. begin_server_side

Server Side
===========

On the **server-side**, the Oro application is composed of the multiple systems and elements that interact to deliver a reliable, scalable, and responsive Oro solution. They are detailed in the following sections.

.. contents:: :local:
   :depth: 1

Oro PHP Application
-------------------

The core component, let us call it **Oro PHP Application**, is a modular **PHP** web application that leverage the **Symfony** framework and **Doctrine ORM** strengths. It interacts with the following system components:

* Database and RDBMS
* Web Server and PHP
* Message Queue
* Search Engine

Database and RDBMS
------------------

**Oro application** uses the **database** to store application data and uses Doctrine database abstraction layer (DBAL) and object-relational mapper (ORM) to interact with the database. That enables out of the box support of various databases enabled by Doctrine. On top of that, in the Oro application, Doctrine capabilities are extended with additional database functions in the `Oro Doctrine Extensions <https://github.com/oroinc/doctrine-extensions>`_ library. Currently the extended functions are supported for MySQL and PostgreSQL databases only.

Supported RDBMs:

* MySQL in CE and EE
* PostgreSQL in EE only (for scalability)

.. note:: For implementation details, see :ref:`Database System Component <op-structure--database>` topic for more information about the database component.

Web Server and PHP
------------------

A **web server** is an HTTP server that manages client requests and proxies them to the **Oro PHP Application**.
**Web server** may rely on the **PHP-FPM** to process requests to **Oro PHP Application** and prepare the response.

Supported web servers: `Apache <https://httpd.apache.org/docs/>`_ and `Nginx <https://www.nginx.com/resources/wiki/>`_

Message Queue
-------------

Oro application uses **Message Queue** to enable asynchronous processing for the heavy jobs that, when executed immediately, may cause performance degradation. For example, reindexation of a large volume of data, creation of large bulks of items, etc. is usually handled via MQ consumers. To process the queued messages, Oro application uses a proprietary consumer service. It is running as a daemon and handles all the asynchronous jobs (messages) registered within a Message Queue. Consumer service is scalable and can run as a parallel processes and/or on multiple servers to handle a large volume of asynchronous processes. Number of processes required depends on the server capacity. To guarantee the acceptable response time and address spikes in the server-side workload, you can scale the message processing by adding more consumer services on demand.

Supported MQ solutions:

* Proprietary DB-based MQ in CE and EE
* RabbitMQ in EE only (for scalability)

.. note:: For implementation details, see :ref:`Message Queue <op-structure--mq--index>` topic for more information about the message queue component.

Search Engine
-------------

Oro application uses **Search Index** to enable full-text search and speed up the run-time access to the large amounts of application data.

Supported search index providers:

* :ref:`DB full-text search <search_index_db_from_md>` in CE and EE
* :ref:`Elastic Search <elastic-search>` in EE only

.. note:: For implementation details, see :ref:`Search Index <search_index_overview>` topic for more information about the search index component.

Notes on Deployment Options
---------------------------

For a compact and resource-efficient deployment, all systems and elements of the Oro application may be hosted on a single physical or virtual server instance.
For scalable high-load deployments:
Multiple instances of Oro application may be hosted on their dedicated web servers, where the load balancer directs client requests to the necessary web server.
All systems and elements of the Oro application may be hosted on their own dedicated server and could be scaled separately.

.. stop_server_side

**Next step**: :ref:`Oro PHP Application Structure <architecture-oro-php-application-structure>`