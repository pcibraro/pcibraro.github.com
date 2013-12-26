---
layout: post
title: "Security Token Handlers in Geneva Framework"
date: 2009-02-04
comments: true
categories: Geneva
---

According to the Geneva documentation,

*"SecurityTokenHandler defines an interface for plugging custom token
handling functionality. Using the SecurityTokenHandler you can add
functionality to serialize, de-serialize, authenticate and create and
specific kind of token"*

I can see dead people ..... :)

![Six](/images/legacy/Six.jpg)

Haven't we seen this before ? Oh, yes, I think we did. The token
managers in WSE, they are pretty much the same thing. It looks like the
Geneva team came up with a solution that worked well in the past with
WSE. One token manager for each kind of token we want to consume in our
application. If your app needs to consume a custom token or customize an
existing one, just derive the SecurityTokenHandler base class or one of
the existing SecurityTokenHandler implementations and override some of
its methods with custom functionality. For instance, the Geneva
Framework now comes with two built-in token handlers for Username
tokens, a MembershipUsernameSecurityTokenHandler for validating users
against a membership provider and a WindowsUsernameSecurityTokenHandler
for doing the same against a windows account store.

Most of the code we had in the past as part of authorization policies
(IAuthorizationPolicy) for mapping claims or validating tokens in a
UsernamePasswordValidator or X509CertificateValidator has now moved to
token handlers in the Geneva framework.

I like this way of extending a custom handler for supporting new kind of
tokens, it is quite more straightforward to me than the model currently
supported by WCF.

