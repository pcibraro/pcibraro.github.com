---
layout: post
title: "A bad idea, EF Entities over WCF Part II"
date: 2009-07-13
comments: true
categories: .NET
---

In the previous post, I discussed how bad was to expose EF entities
directly as data contracts from a perspective of “interoperability”.
This also revealed a lot of internal implementation details about EF
that the client applications should not worry about. 

As [Daniel Simmons](http://blogs.msdn.com/dsimmons/) pointed out, the
next version of Entity Framework (4.0) will ship with a new and very
useful feature to represent the data entities using POCOs (Self-tracking
entities). This new support for POCOs will make possible to simplify a
lot the final representation of entities on the wire, hiding all those
annoying details about EF.  You will also able to decorate the entities
with DataContract/DataMember attributes if you do not want to expose the
whole entity in the service.

However, I still do not buy the idea of using the EF entities as data
contracts. By doing that, you are coupling your service interface to the
final data model, which is not a good idea since both layers usually
evolve at different rates. What is worse, any change in the database
model might affect your service interface as well, breaking the contract
that any potential consumer have with the service.

This last aspect is related to versioning. The versioning between the
two layers is completely different, you might have for instance an User
entity in the data model, and different versions of the same entity in
the service interface, UserV1, UserV2, etc (Any of these corresponding
to a different version of the service). The service is the responsible
of making the transition between the different data contracts to the
final data model entities.

I think this might be only useful for RIA applications where you use the
services as a mechanism to interchange data between your backend layer
and the UI, for instance web applications that use Ajax Callbacks or
Silverlight applications. So, the number of potential consumers of the
services is really low.

