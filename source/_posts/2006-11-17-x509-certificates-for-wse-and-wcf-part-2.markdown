---
layout: post
title: "X509 Certificates for WSE and WCF - Part 2"
date: 2006-11-17
comments: true
categories: .NET
---

I am writing this post as an extension to the previous one, ["Creating
X509 Certificates for WSE or
WCF"](/cibrax/archive/2006/08/08/Creating-X509-Certificates-for-WSE-or-WCF.aspx)

I lately received some feedback from a colleague Albert, and I think
it is worth mentioning.

Albert came out with a common dilemma nowadays, "how to buy a X509
certificate for his application to a well know certificate authority".

The main problem here is that these vendors only sell closed products
such as e-mail certificates, or SSL certificates, they hardly know about
the requirements for WSE (I guess, WCF should have similar
requirements).

According to what Albert found out, a certificate for WSE should have
the following attributes:

**KeyUsage: Digital Signature, Non-Repudiation, Key Encipherment, Data
Encipherment (f0)**

**Enhanced Key Usage: Client Authentication (1.3.6.1.5.5.7.3.2)**

**this one seems to be optional - Server Authentication
(1.3.6.1.5.5.7.3.1)**

But, most SSL certificates have the following properties:

**KeyUsage: Digital Signature, Key Encipherment (a0)**

**Enhanced Key Usage: Server Authentication (1.3.6.1.5.5.7.3.1) Client
Authentication (1.3.6.1.5.5.7.3.2)**

Which explains why a normal SSL certificate can not be used by WSE (and
you receive the "Certificate does not support data encryption" error
message).

As a result, knowing those requeriments, Albert could finally buy the
right certificate to one of the well-know vendors. (Sorry, I will not
add any marketing stuff here). So, if you ever need to buy a certificate
for WSE or WCF, ask for those certificate characteristics to avoid any
problem in advance.

