---
layout: post
title: "WS-Federation quickstart for WSE 3.0"
date: 2006-06-14
comments: true
categories: ASP.NET
---

Microsoft has recently released a WS-Federation sample based on the SAML
implementation for WSE 3.0.\
This sample adds some new cool features to the SAML implementation and
shows a scenario similar to what I described a couple of months ago in
this [post](/cibrax/archive/2006/02/02/437180.aspx "post").

These are some of the features provided in this sample,

​1. SAML token encryption based on a configuration setting. This setting
basically allows encrypting the attributes\
in the token using a secret key shared between the STS and the service.\
2. SAML authorization assertions. In addition to the common attribute
assertions, the SAML token now supports authorization assertions, which
are useful to describe permissions over different resources (For
example, the owner of this token is allowed to read the financial
files). The quickstart also provides an WSE authorization assertion to
enforce some of these permissions in the SAML token.\
3. Custom SAML token managers. These token managers were specifically
designed for this sample, but they are a good starting point if you want
to know more about the different approaches to customize a SAML token
manager.

Download the sample from this
[location](http://www.microsoft.com/downloads/details.aspx?familyid=630ad3a5-05ca-4c24-a50a-9e7007323ec2&displaylang=en "location") 

Enjoy it :)

