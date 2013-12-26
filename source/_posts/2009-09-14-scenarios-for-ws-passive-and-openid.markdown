---
layout: post
title: "Scenarios for WS-Passive and OpenID"
date: 2009-09-14
comments: true
categories: .NET
---

I was wondering these days what would be the point in using WS-Passive
when there is another simple sign-on solution, OpenID, that works really
well and it’s getting a great adoption in the community. I can not say
the same about WS-Passive, I haven’t seen any concrete implementation
yet (For instance, Microsoft is planning to release a first
implementation as part of the WIF framework before the end of this
year).

However, I reached the conclusion that both technologies are prepared
for addressing different scenarios. For me, OpenID is more suitable for
community or social networking web sites that expect to have a great
number of users, and can benefit from using the large user databases
that some of the existing OpenID providers already have. This is also a
great thing for the final user as he can reuse his public OpenID
identifier to log into all these web sites. 

On the other hand, WS-Passive seems to work better for enterprise
scenarios in companies that might have some sort of security
infrastructure in place (Customer user databases, active directory,
active STSs for services) and do not want to have third parties (other
than the parties involved in the business) to participate in the
business transactions or to share their user databases with these
parties. This is closely related to one of the identity laws that Kim
Cameron [announced a time
ago](http://www.identityblog.com/stories/2004/12/09/thelaws.html). Law
\#3, “Justifiable Parties”, *Digital identity systems must be designed
so the disclosure of identifying information is limited to parties
having a necessary and justifiable place in a given identity
relationship.*

This was one of the big failures in the adoption of Passport as
Single-Sign-On technology. Some relying parties did not understand the
purpose of having Microsoft (and Passport) involved in their business
transactions.

In addition, WS-Passive only specifies a way to interchange a security
token with user claims between the involved parties (Client, STS and
Relying Party), so it provides the flexibility of using any security
token profile. The most common token profile used today is SAML, as it
can be customized with custom assertions or claims specific to the
business. OpenID also specifies a way to exchange custom data (other
than [Simple
Registration](http://openid.net/specs/openid-simple-registration-extension-1_0.html)
or SReg data) through the [“Attribute
Exchange”](http://openid.net/specs/openid-attribute-exchange-1_0.html)
spec. However, it looks like many of the existing relying parties or
identity providers do not support this spec as it is discussed
[here](http://dennisbloete.de/blog/on-openid-attribute-exchange/).

I am just guessing :). What do you think ?. \


