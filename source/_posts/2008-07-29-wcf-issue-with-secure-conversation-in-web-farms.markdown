---
layout: post
title: "WCF - Issue with Secure Conversation in Web farms"
date: 2008-07-29
comments: true
categories: WCF
---

While I was working for one of my customers, we ran into a very strange
problem when they tried to deploy some WCF services in a web farm using
cookie sessions (In order to enable secure conversation in this kind of
scenario, cookies must be used to track the state of the Secure
Conversation Tokens).

At some point, we start receiving a random exception with the following
message,

***System.ServiceModel.Security.MessageSecurityException: Message
security verification failed. ---&gt; System.Xml.XmlException: There was
an error deserializing the security token XML. Please see the inner
exception for more details. ---&gt; System.ArgumentException: The
SecurityContextSecurityToken with
context-id=urn:uuid:502e208e-8db2-4814-8623-d3050273e875 (no key
generation-id) has expired***

Later, we found out that a time skew setting was not supported in WCF
for the SCT cookies, and the error was happening because of an slight
difference in time between the servers in the farm.

Fortunately, [Brent Schmaltz](http://blogs.msdn.com/brentschmaltz) has
made public a hot fix for that problem in his blog. You can find the
complete solution and a better description of the problem in [this
post](http://blogs.msdn.com/brentschmaltz/archive/2008/06/20/cookie-sessions-and-wcf.aspx).

Enjoy.



