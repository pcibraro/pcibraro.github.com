---
layout: post
title: "Security Identity propagation for WCF Ajax endpoints in ASP.NET"
date: 2010-10-21
comments: true
categories: .NET
---

A UI driven service is usually a service implementation that only makes
sense in the context of the UI for solving an specific use case, and not
something that you might want to share or expose to third parties.
Typical examples of UI driven services are AJAX endpoints, which you
build for supporting partial updates in a page. The implementation of
this kind of services can take the form of a simple http endpoints,
which could adhere to the REST principles or not, or SOAP web
services.   

As the AJAX endpoints are consumed by the web browser on behalf of the
user, you will typically want to propagate the web browser security
context to this service, and not a new one, so the web pages and the
services both run under the same user identity.

If the web pages and the services are both running in the same hosting
stack like ASP.NET, this does not represent a problem at all, as they
both share the security context of the host. For example, if you
implement the services as ASMX web services, or Http Handlers, or MVC
controller actions, they will all share the ASP.NET security context
with the pages.

WCF runs by default in its own hosting space, which is not dependant of
ASP.NET, so here is where the problem begins. Message security is
obviously discarded for this scenario, as a client script does not know
how to handle cryptographic material for doing all the message signing
and encryption. In addition, it would add some unnecessary complexity to
the solution, which is not need for the scenarios that an AJAX endpoint
tries to achieve. Therefore, transport security is the right choice for
WCF Ajax endpoints if you want to encrypt the traffic with SSL, and none
if the information does not need to be encrypted because it is not
sensitive. In both cases, the right choice for a binding is
“basicHttpBinding” for SOAP services and “webHttpBinding” for any other
Http endpoint that does not use soap envelopes.

For example, the following binding configures a SOAP service with
“basicHttpBinding” and no security.

\<basicHttpBinding\> \
        \<binding name="AjaxEndpoints"\> \
          \<security mode="None"\>\</security\> \
        \</binding\> \
\</basicHttpBinding\>

In addition, as you want to propagate the user identity from ASP.NET to
the WCF services. You need to enable the ASP.NET compatibility mode in
the service, so ASP.NET and the WCF Ajax services both share the same
user identity, no matter which security mechanism was configured in
ASP.NET (forms, claims, or any http authentication mechanism).

For enabling the ASP.NET compatibility mode, you have to add the
following configuration to the serviceModel section,

~~~~ {#codeSnippet style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
<system.serviceModel>    <serviceHostingEnvironment aspNetCompatibilityEnabled="true"/>
~~~~

\
And decorate your service with the AspNetCompatibilityRequirements
attribute,

~~~~ {#codeSnippet style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
[AspNetCompatibilityRequirements(RequirementsMode=AspNetCompatibilityRequirementsMode.Allowed)]public class Service1 : IService1
~~~~

