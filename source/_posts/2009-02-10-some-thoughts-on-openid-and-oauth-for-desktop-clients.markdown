---
layout: post
title: "Some thoughts on OpenID and OAuth for Desktop clients"
date: 2009-02-10
comments: true
categories: OAuth
---

OpenID and OAuth are today excellent solutions for "Single Sign On"
(SSO)  and "Authorization Delegation" respectively. They are, however,
based on Http Redirections and therefore, tied to passive clients or
commonly called web browsers.

An interesting research was made by google some time ago, it can be
found
[here](http://sites.google.com/site/oauthgoog/UXFedLogin/desktopapps).
After reading that article, it looks like they could not get rid of a
browser at all :(.

If that does not work for you, another solution could be WS-Federation
Active Profile. 

"SSO" is an inherent feature of WS-Federation, not doubt about it.

"Authorization Delegation" can also be emulated with a combination of
"SSO" and authorization claims. In this scenario, we always give our
credentials to an identity provider we trust, there is no need to give
away our credentials to any site or service involved in a transaction.
The authorization claims also represent fine-granular permissions of
what we are allowed to do on the service side, and again, they can
provided by identity provider itself or a resource STS. I discussed this
approach in my last post, ["Addressing Authorization with OAuth or the
.NET Access Control
Service"](http://weblogs.asp.net/cibrax/archive/2009/02/06/addressing-authorization-with-oauth-or-the-net-access-control-service.aspx),
the resource STS in this case would be the ACS service.

