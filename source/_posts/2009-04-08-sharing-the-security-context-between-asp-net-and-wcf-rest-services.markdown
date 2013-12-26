---
layout: post
title: "Sharing the security context between ASP.NET and WCF REST Services"
date: 2009-04-08
comments: true
categories: REST
---

It is very common for WCF services that work as Ajax callbacks and
ASP.NET pages that live in the same web application to share a common
security context for the authenticated user. However, in order to make
this happens, the [ASP.NET compatibility
mode](http://blogs.msdn.com/wenlong/archive/2006/01/23/516041.aspx) must
be enabled for the WCF service. When that setting is enabled, WCF
basically includes the service call within the ASP.NET pipeline, all the
ASP.NET modules configured for the application are executed, and as
result, the HttpContext get initialized for that service as it was a
normal page. 

Optionally, an IAuthorizationPolicy can be used to inject the
HttpContext.User into the WCF security context, and completely decouple
in that way, the service implementation from the Http security context.

For example, we can have a web application configured with Forms
Authentication or the new FederationModule in geneva framework for
authenticating users with passive federation, and share that security
context with the services or ajax callbacks that the web pages consume.

For this scenario, the WCF REST Starter kit already includes an
IAuthorizationPolicy implementation for initialization the WCF security
context with the authenticated user in the web application. That
authorization policy is automatically injected in the service execution
pipeline by the WebServiceHost2 host shipped also within the kit. If you
are using that WCF host, you do not have to do anything extra other than
enabling the ASP.NET compatibility mode for the services.

Configuring the ASP.NET compatibility mode requires two steps.

​1. Add a configuration settings in the configuration file,

\<system.serviceModel\>

\<serviceHostingEnvironment aspNetCompatibilityEnabled="true" /\>

\</system.serviceModel\>

 2. Add support for this mode in the service implementation. This is
done through an special attribute “AspNetCompatibilityRequirements”
added to the class definition

[AspNetCompatibilityRequirements(RequirementsMode =
AspNetCompatibilityRequirementsMode.Required)] 

[ServiceContract]

public class ClaimsService

Once you have done that, the user principal will become available to the
service through the WCF security context or the Thread.CurrentPrincipal
property.

