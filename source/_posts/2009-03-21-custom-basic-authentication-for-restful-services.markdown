---
layout: post
title: "Custom Basic Authentication for RESTful services"
date: 2009-03-21
comments: true
categories: REST
---

It’s very common when developing RESTful services to authenticate users
against a proprietary user database. This is generally done with a
combination of username and password through http basic authentication.
Unfortunately, basic authentication is tied to windows accounts in IIS,
which leads us to find out some alternatives or workarounds to support
this scenario. WCF 3.5 made possible to authenticate transport
credentials with one of the existing UsernamePasswordValidator
extensions, however, this approach does not work for IIS hosted
services.

[Dominick](http://www.leastprivilege.com/CommentView.aspx?guid=f9453fb0-6e2a-4faf-8cf9-62162dc7531e)
solved this problem with a module plugged directly in the ASP.NET
pipeline that works like a charm, but it requires some additional WCF
settings and a custom IAuthorization policy to flow the user principal
to the WCF service instance. His solution works for ASP.NET applications
as well.

This problem can be also solved at a deeper level in the WCF transport
model using a message interceptor. The message interceptor can receive a
traditional Membership provider in the constructor class, and use it
later for authenticating the users. In addition, this message
interceptor can also automatically pass the user credentials to the WCF
service instance.

As any other message interceptor, it can be configured directly in the
WCF service factory with no need of having additional configuration.
This can be easily done in the “svc” file hosted in IIS.

\<%@ ServiceHost Language="C\#" Debug="true" Service="Service"
Factory="AppServiceHostFactory" %\>

class AppServiceHostFactory : ServiceHostFactory

{

   protected override ServiceHost CreateServiceHost(Type serviceType,
Uri[] baseAddresses)

   {

     WebServiceHost2 result = new WebServiceHost2(serviceType, true,
baseAddresses);

     result.Interceptors.Add(new BasicAuthenticationInterceptor(

                System.Web.Security.Membership.Provider, "foo"));

     return result;

   }

}

BasicAuthenticationInterceptor is the message interceptor I built for
this post, it receives the MembershipProvider and a default realm that
will be returned together with an 401 http error (Unauthorized) in case
the user was not correctly authenticated.

The example is available to download from
[here](/images/legacy/BasicAuthentication.zip).

