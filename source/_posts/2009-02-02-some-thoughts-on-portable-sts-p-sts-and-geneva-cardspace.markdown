---
layout: post
title: "Some thoughts on Portable STS (P-STS) and Geneva Cardspace"
date: 2009-02-02
comments: true
categories: Federation
---

The other day and friend of mine asked me about portable STS
implementations, if I knew about any available solution that he could
use on his company. That reminded me of a conversation I had like two
years ago with another developer working on custom .NET CLR framework
version for portable devices (like smartcards). As part of that project,
his team was also working on a TCP/IP communication stack for the
device, and a http handler for accepting raw WS-TRUST messages. One goal
for that project was to have a P-STS that could be interoperable with
WCF. The idea seemed very promising at time.

So, what is a PSTS after all ?. In a few words, it is a service running
on a portable device that exposes WS-TRUST endpoints and can issue
security tokens of any kind (e.g, SAML tokens).

Making a search today on
[google](http://www.google.com.ar/search?hl=en&q=Portable+Security+Token+Service)
will drop several P-STS products or solutions,  some of them also claim
to be interoperable with WCF and Microsoft Cardspace V1.

In terms of identity management, A P-STS really makes a great different
over existing authentication mechanisms like username/password, X509
certificates or any other kind of two-factor authentication device. Most
of these authentication mechanisms are widely accepted and used today in
applications within corporate environments or applications that requires
off-line support. However, sometimes they lack of a truly identity
support, which means that they do not represent the user identity at all
in the context of those applications, they are just a way of identifying
returning users, or they are hard to extend with additional user's
identity claims.

I can not deny that X509 certificates have demonstrated to be a very
effective and secure way to authenticate users. In addition, X509
certificates can be extended with some custom attributes, the space is
limited, but at least there is a possibility. However, X509 certificates
represent hard tokens, the claims stored on a certificate can not be
changed once it has been issued. Therefore, they are a good solution as
long as their information do not change frequently over a period of
time.

Issue tokens (e.g SAML tokens) on other hand are more dynamic and
cheaper to create. They usually have a short expiration time, they can
issued and used  on the fly, but what is more important, they can carry
custom information or claims about the subject it has been issued for.

Some good news is that the Geneva Cardspace team has also announced some
support for roaming scenarios in Cardspace V2. There will be a way to
store our identity cards on a device (or somewhere in the cloud), which
will be great to combine with a P-STS, no need to export/import the
cards anymore. This scenario was not possible in Cardspace V1, and [here
is the
explanation](http://blogs.msdn.com/vbertocci/archive/2008/01/27/on-the-idea-of-portable-sts-p-sts.aspx).
According to what Rich Randall mentioned in the PDC talk "BB44 Identity:
Windows CardSpace "Geneva" Under the Hood ", the future Cardspace
interface could look as follow,

 ![](/images/legacy/Infocard.jpg)

 

As you can see, it will not be long until we have complete and portable
identity solutions for roaming scenarios.

