---
layout: post
title: "Hermes – A new open source alternative for doing pub/sub over Http"
date: 2011-07-12
comments: true
categories: .NET
---

After a few months of collaborative work across several members in our
Tellago crew, Hermes is finally
[here](https://github.com/TellagoDevLabs/Hermes/). Hermes, also known as
the great messenger of god in the Greek mythology, was the name we gave
to this new open source alternative for doing durable pub/sub messaging
over http.

There is no doubt that MSMQ has been the predominant technology for
doing Pub/Sub within the Microsoft stack given the reliability it
provides as a temporary storage for messages. However, the main problem
of using a queue technology like that is that you sacrifice
interoperability for better reliability, and interoperability usually
matters a lot in the enterprise. How many times you have heard stories
of people doing crazy things in the enterprise with queue connectors or
similar technologies for decoupling systems implemented in different
platform stacks.

Http is something universal accepted in all the existing platforms and
makes the things really simple in that sense. Almost all the existing
programming languages in the world provide libraries for consuming Http
endpoints or Web Apis with a few lines of code.

There are some Pub/Sub alternatives using Http and RESTful services in
the cloud like the AppFabric Service Bus or PubNub, but Hermes is also
suited to be used within the boundaries of a single organization.  Every
time you want to decouple two systems or provide valuable business
events to other systems you don't know upfront, Hermes becomes a great
candidate for implementing that kind of scenario.

As I said before, you might sacrifice reliability in the sense that you
are using Http for publishing the events into the Hermes repository, and
a http channel must be available at that time, but that’s an issue on
the publisher side only. The subscription side is implemented through
simple http polling with Atom, so the subscriber can read the messages
when an Http channel is available. Also, all the transformation
responsibilities are moved to the subscriber side simplifying a lot the
core messaging engine in Hermes.

Hermes uses MongoDB as the backend storage for the messages, making
extremely easy to store, partition and index high volumes of messages
for a large number of topics and subscriptions.

You can read more details about the underline architecture in [the
announcement](http://weblogs.asp.net/gsusx/archive/2011/07/12/tellago-announces-hermes-a-publish-subscribe-messaging-engine-based-on-http-and-mongodb.aspx)
made by Jesus.

Enjoy!!.

