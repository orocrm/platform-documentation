.. _dev-guide-system-websockets:

WebSockets
==========

**WebSockets** is a full-duplex communication protocol for real-time messaging between a server and clients through
persistent connections.

The main aim of WebSockets is to provide real-time notifications for clients about server events or changes without a
need for the client to repeat requests to the server for new information. For example:

* When someone changed the document another user was in progress of editing. In this case, a notification informing that someone is working with this document or that the document has been modified is immensely helpful.

* Real-time charts of stock prices or currency exchange rates on financial portals. The accuracy and timeliness of this type of data is crucial for visitors of the portal and manual refreshment of the page can be very exhausting.
* Real-time instant messaging on the website. Users must receive messages without refreshing the chat page. 
* etc.

.. _dev-guide-system-websockets-architecture:

How It Works
------------

In Oro applications, WebSocket communications are built using
`Web Application Message Protocol (WAMP) <https://wamp-proto.org/>`_, a WebSockets subprotocol aimed at
organizing the communication between program components in the applications with a loosely coupled architecture.

The main two parts of WAMP protocol are `Remote Procedure Call <https://en.wikipedia.org/wiki/Remote_procedure_call>`_
(RPC) mechanism and `PubSub <https://en.wikipedia.org/wiki/Publish%E2%80%93subscribe_pattern>`_ messaging pattern.

**RPC** mechanism allows calling a function from a different code remotely via a WebSocket.

**PubSub** messaging pattern implies that when messages are published to topics by publishers (or in other words “channels”), the broker distributes them to clients that are subscribed to these topics.

Therefore, the **WAMP** protocol implies that there is a **WebSocket server** that plays a role of message broker; there are ways for the
application components to **register topics** for messages, **publish messages** to topics and **subscribe to topic** messages.

In Oro applications, all WebSocket-related functionality is provided by
`OroSyncBundle <https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/SyncBundle/>`_. As OroSyncBundle is
part of OroPlatform, which is the base for all Oro applications, the WebSockets functionality exists in all Oro
applications.

.. note:: WebSockets functionality exists only in the admin UI of Oro applications which allows for guaranteed authentication of all clients
    who subscribe to the topic messages.

Getting Started
---------------

You need to :ref:`Setup and Configure <dev-guide-system-websockets-setup-configuration>` the Websockets functionality before you can start using it in Oro applications.

Out-of-the-box, OroSyncBundle uses WebSockets connection for two purposes:

* :ref:`Content outdating notifications <dev-cookbook-system-websockets-content-outdating-notifications>` --- To provide flash notifications for the user informing about outdated content, if several users try to edit the same entity record simultaneously.
* :ref:`Maintenance mode notifications <dev-cookbook-system-websockets-maintenance-mode>` --- To send flash notifications to all application site visitors once a developer turns on the system maintenance mode by a console's CLI tool.

To start use websockets messages for your custom functionality, please, refer to the articles
:ref:`Create Your Own Topic for Publishing and Subscribing <dev-cookbook-system-websockets-create-topic-and-handler>` and
:ref:`Publish Messages to Existing Topics <dev-cookbook-system-websockets-publish-to-topic>` for the guidance.

Related Cookbook Articles
-------------------------

* :ref:`Use Authentication and Authorization in WebSockets Connections <dev-cookbook-system-websockets-authentication-autorization>`
* :ref:`Use Content Outdating Notifications in Oro Applications <dev-cookbook-system-websockets-content-outdating-notifications>`
* :ref:`Use Maintenance Mode Notifications in Oro Applications <dev-cookbook-system-websockets-maintenance-mode>`
* :ref:`Publish Messages to Existing Topics <dev-cookbook-system-websockets-publish-to-topic>`
* :ref:`Create Your Own Topic for Publishing and Subscribing <dev-cookbook-system-websockets-create-topic-and-handler>`

.. toctree::
    :hidden:
    :titlesonly:
    :maxdepth: 2

    setup_and_configuration

