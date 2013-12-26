---
layout: post
title: "Addressing authorization with OAuth or the .NET Access Control Service"
date: 2009-02-06
comments: true
categories: .NET-Services
---

**OAuth**

As I mentioned in the post, ["OAuth Channel for REST
services"](http://weblogs.asp.net/cibrax/archive/2008/11/14/oauth-channel-for-wcf-restful-services.aspx),
OAuth allows a client application to obtain user consent (as access
tokens) for executing operations over private resources on his behalf.
Resources in this context represent anything from the user that the
service provider make public through services, they could be for
instance contacts, pictures or personal information to name a few.

The access token that the consumer gets from the service provider
represents in some way an Access Control List that maps directly with
permissions granted by the user over his resources. For example, John
provides read/write access to his contacts on Windows Live (Service
provider) to a third party service (consumer).

A previous direct trust relationship must exist between the consumer and
the service provider in order to make all this happen, that relationship
in OAuth also takes the form of a Request Token. If the consumer can not
get a request token from the service provider, the user is not even
redirected to this last one for negotiating the access token.

As more service providers get involved in a simple scenario, more access
tokens the consumer will have to negotiate. In addition, the user should
have registered in each one of those service provider prior to use the
consumer application, unless he was  lucky enough to have Open ID
authentication in some of those services.

![](/images/legacy/OAuth1.jpg)

![](/images/legacy/OAuth2.jpg)

In the image above, the user is authenticated by the service provider
during the request token/access token exchange.

**.NET Access Control Service**

The Microsoft .NET Access Control Service was recently announced in the
PDC as part of the Windows Azure platform. (It was formerly part of
Biztalk services). Today, it is complemented by two other services, the
Microsoft .NET Service Bus and Microsoft .NET Workflow Service.

This service in addition to be a valid Secure Token Service (WS-Trust)
that can participate in the identity metasystem, it is a claim
transformer. It was conceived with the idea of mapping some input claims
(Identity claims usually) into authorization claims that represent an
ACL for the service running on the relying party.

The SAML token containing these output claims would be equivalent to the
access token in OAuth. 

As we saw in OAuth, a prior trust agreement must exist between the
relying party and the .NET Access Control Service. In this case, A X509
public key for the relying party must be registered on the .NET ACL
service (for encrypting the SAML token), and the relying party must have
a X509 public key coming from the .NET ACL for verifying the token
signature. As part of this agreement, also some control rules for
mapping claims must defined  in the .NET ACL service configuration (The
.NET ACL will use the appliesTo header in the WS-Trust RST message to
determine which rules have to be used).

If we analyze now the scenario discussed before with OAuth, a single
consumer and multiple service providers, the client always authenticates
against the same identity provider, no matter the number of the service
providers involved, which is a pretty good thing. OAuth depends on Open
ID for getting the same effect.

The picture below show the complete scenario with a single relying
party,

 ![](/images/legacy/ACL1.jpg)

 

