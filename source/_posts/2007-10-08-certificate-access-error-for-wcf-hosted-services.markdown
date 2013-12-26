---
layout: post
title: "Certificate Access Error for WCF hosted services"
date: 2007-10-08
comments: true
categories: WCF
---

*The problem appears when a WCF Service hosted in an IIS tries to load a
certificate from the Windows Certificates Store with the account of the
Application Pool where the service runs, and the account’s profile is
not previously loaded. When a user logs on interactively, the system
automatically loads the user's profile. If a service or an application
impersonates a user, the system does not load the user's profile.
Therefore, the service or application should load the user's profile
with LoadUserProfile.*

*When this happens the operation throws the following exception:*

*System.Security.Cryptography.CryptographicException: The system cannot
find the file specified. [VIA [Mariano Sanches
Blog](http://marianosz.blogspot.com/)]*

 

The problem is basically related to the method that WCF uses to encrypt
the Secure conversation cookie. By default, WCF encrypts the cookie
using DPAPI (Current User), so the user profile should be loaded in
order to use that encryption method.

Mariano mentioned one of the workarounds, which requires to load
the user's profile of the account configured in the IIS Application pool
using some windows API's.

Another workaround is to change the method used by WCF to encrypt
the SCT cookie and eliminate the DPAPI dependency. An advantage of this
method is that it works fine for Web Farm scenarios.

Fortunately, WCF provides an extension to change the method used to
encrypt and serialize the SCT Cookie. This extension is a
SecurityStateEncoder and it can be plugged-in by code in the service
host:

**serviceHost.Credentials.SecureConversationAuthentication.SecurityStateEncoder**

You can find an excellent example about how to implement a custom
SecurityStateEncoder
[here](http://wcf.netfx3.com/files/folders/authorization/entry11436.aspx),
(WCF.Netfx3.com web site, WS-SC With State Encoder)

