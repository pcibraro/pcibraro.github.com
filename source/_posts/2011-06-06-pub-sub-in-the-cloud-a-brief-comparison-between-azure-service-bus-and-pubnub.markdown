---
layout: post
title: "Pub/Sub in the cloud–A brief comparison between Azure Service Bus and PubNub"
date: 2011-06-06
comments: true
categories: .NET
---

Publish/Subscribe in the cloud has became relatively important lately as
an integration pattern for business to business scenarios between
organizations. The major benefit of using a service hosted in the cloud
as intermediary is that publishers and subscribers don’t need to be
publicly addressable, be in the same network  or be able to talk each
other directly. The cloud infrastructure allows this intermediary
service to scale correctly as the number of publishers or subscribers
increase, and also to act as a firewall for brokering the communication
(Publishers or subscribers need explicit permissions to connect, send or
receive messages from the intermediary service).

This pattern can  be used in workflow systems to relay events among
distributed computer applications, update data in business systems or as
a way to move data between data stores. For example, in an order
processing application, notifications must be sent whenever a
transaction occurs; an order is placed in a system, the order details
are forwarded as a message to a payment processor service for approval,
and finally, an order confirmation message is sent back to the system
where the order was originally created.

[![pubsub](http://weblogs.asp.net/blogs/cibrax/pubsub_thumb_5A6289CA.jpg "pubsub")](http://weblogs.asp.net/blogs/cibrax/pubsub_478E0620.jpg)

This infrastructure typically supports the idea of “topics” or named
logical channels. Subscribers will receive all the messages published to
the topics to which they subscribe, and all subscribers to a topic will
receive the same messages.

I am going to discuss today two available solutions in the cloud, the
“AppFabric Service Bus”, which is part of the Microsoft PaaS cloud
strategy known as Azure and also a relatively new implementation
“PubNub” hosted in the  Amazon EC2 cloud infrastructure.

AppFabric Service Bus
---------------------

The AppFabric Service Bus is a service running in Microsoft data
centers. This service acts as broker for relaying messages through the
cloud to services running on premises behind network obstacles of any
kind, such as firewalls or NAT devices. The Service Bus secures all its
endpoints by using the claim based security model provided by the Access
Control service (Another service available as part of Azure AppFabric).
You can find a lot of interesting features as part of the service bus
such as federated authentication for listening or sending to the cloud, 
a naming mechanism for the endpoints in the cloud, a common messaging
fabric with a great variety of communication options, or a discoverable
service registry that any application trying to integrate with it can
use. \
In the first release, the service bus originally provided a relay
service for integrating on-premises applications with services running
on the cloud. At that time, the integration with the relay service could
be done in two ways, a message buffer in the cloud accessible through a
REST API or using the traditional WCF programming model with special
channels talking to the relay service on the cloud. By using the WCF
programming model, the interaction with the relay binding was almost
transparent for applications, as all the communications details were
handled at channel level by WCF.  This message buffer was a temporary
store for the messages, so they disappeared after being consumed or when
they expired.

The AppFabric team recently announced the availability of a new feature
for supporting durable messaging at the service bus level. Durable
messaging in this context comes in two flavors,  reliable message
queuing and  durable publish/subscribe messaging. The main difference
between them is the number of parties that can consume a message
published in the service bus. While a message is consumed by a single
party when a queue is used, the publish/subscribe model relies on
topics, which allows multiple parties to subscribe to the messages
received in an specific topic (Every party receives a copy of the
message basically).

The pricing model for the service bus is currently based on the number
of used connections. Every message sent to the service bus usually
involves two connections, one connection for sending or publishing the
message and another connection for receiving it (this might change for
the model where you have multiple subscribers for a message). [This
thread](http://social.msdn.microsoft.com/Forums/en-US/netservices/thread/11d687db-7b3f-4148-8e81-48a8290f8146/)
in the MSDN forums discusses the model in details, and I have to admit
it takes some time to digest.

Advantages

-   The service bus supports a good isolation level based on service
    namespaces.  A service namespace represents a level of isolation for
    a particular set endpoints, and you can associate multiple service
    namespaces to an Azure account. For example, you can have two
    different applications associated to your Azure account and each one
    them listening on a different service namespace address.
-   The great number of communication options you can find as part of
    the service bus.
-   The REST API, the .NET APIs and the WCF bindings makes the service
    bus really easy to use from any application.

Disadvantages

-   The pricing model is to complex to understand and it is hard to
    predict. Microsoft does not currently offers a good monitoring
    option for determining the number of used connections or predicting
    costs before receiving the monthly bill.
-   The number of service namespaces that you can create in an specific
    Azure account is limited (I believe the number is 50 namespaces, and
    that number can be increased if you make an explicit request). This
    is still a big problem if you want to use the service bus to route
    messages to several machines listening on different namespaces, or
    support a multitenant schema in which a different namespace is
    assigned per tenant. 
-   There is not an API for managing the service namespaces, which
    represents an inconvenient if you want to allocate service
    namespaces dynamically.

PubNub
------

PubNub is a relatively new push service hosted in the cloud. It’s
currently hosted in the Amazon EC2 infrastructure, and provide a set of
APIs for pushing or receiving messages in almost all the languages and
platforms you can imagine. All those APIs are also available as open
source in [GitHub](https://github.com/pubnub/pubnub-api).   While the
main purpose of this service is to serve as a mechanism for pushing data
to different devices (mobile devices, web browsers, etc) via Http, I can
also a find a good use case of this service for pub/sub in the
enterprise.

PubNub pushes data to the different subscribers using a BOSH comet
technology. The idea BOSH comet is to define a transport protocol that
emulates the semantics of a long-lived, bidirectional TCP connection
between a client and a server by efficiently using multiple synchronous
HTTP request/response pairs without requiring the use of frequent
polling or chunked responses.

Subscribers must issue a API call to begin listening for messages on an
specific channel (similar to a topic), automatically keeping the
connection open until the application is closed. Every message sent by a
client application to an specific channel will be forwarded to all the
subscriber listening on that channel.

One the main disadvantages probably is the maximum size for the message
payload that you can send or receive, which is 1.8 Kb (This limit might
be increased, or otherwise, you might need to implement a chunking
channel on your end).

Advantages

-   Extremely fast and easy to use.
-   The pricing model is very easy to understand, you pay for every
    message that you sent basically. This model scales well for a great
    number of clients and servers as well as the price you pay for every
    message is relatively cheap.
-   They offer an API for managing accounts, which is the mechanism they
    use for billing.
-   Client API available in a great number of technologies and
    languages.

Disadvantages

-   The supported message payload size, which is by default, 1.8 kb.
-   They don’t have an exclusive isolation level like the service bus
    does. The only isolation level here is the account.


