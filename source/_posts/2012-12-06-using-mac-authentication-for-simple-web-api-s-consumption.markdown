---
layout: post
title: "Using MAC Authentication for simple Web API’s consumption"
date: 2012-12-06
comments: true
categories: .NET
---

For simple scenarios of Web API consumption where identity delegation is
not required, traditional http authentication schemas such as basic,
certificates or digest are the most used nowadays. All these schemas
rely on sending the caller credentials or some representation of it in
every request message as part of the Authorization header, so they are
prone to suffer phishing attacks if they are not correctly secured at
transport level with https.

In addition, most client applications typically authenticate two
different things, the caller application and the user consuming the API
on behalf of that application. For most cases, the schema is simplified
by using a single set of username and password for authenticating both,
making necessary to store those credentials temporally somewhere in
memory.

The true is that you can use two different identities, one for the user
running the application, which you might authenticate just once during
the first call when the application is initialized, and another identity
for the application itself that you use on every call.

Some cloud vendors like Windows Azure or Amazon Web Services have
adopted an schema to authenticate the caller application based on a
Message Authentication Code (MAC) generated with a symmetric algorithm
using a key known by the two parties, the caller and the Web API.

The caller must include a MAC as part of the Authorization header
created from different pieces of information in the request message such
as the address, the host, and some other headers. The Web API can
authenticate the caller by using the key associated to it and validating
the attached MAC in the request message. In that way, no credentials are
sent as part of the request message, so there is no way an attacker to
intercept the message and get access to those credentials.

Anyways, this schema also suffers from some deficiencies that can
generate attacks. For example, brute force can be still used to infer
the key used for generating the MAC, and impersonate the original
caller. This can be mitigated by renewing keys in a relative short
period of time. This schema as any other can be complemented with
transport security.

Eran Rammer, one of the brains behind OAuth, has recently published an
specification of a protocol based on MAC for Http authentication called
Hawk. The initial version of the spec is available
[here](https://github.com/hueniverse). A curious fact is that the
specification per se does not exist, and the specification itself is the
code that Eran initially wrote using node.js.

In that implementation, you can associate a key to an user, so once the
MAC has been verified on the Web API, the user can be inferred from that
key. Also a timestamp is used to avoid replay attacks.

As a pet project, I decided to port that code to .NET using ASP.NET Web
API, which is available also in github under
[https://github.com/pcibraro/hawknet](https://github.com/pcibraro/hawknet)

Enjoy!.

