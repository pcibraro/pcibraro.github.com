---
layout: post
title: "OAuth channel for WCF RESTful services"
date: 2008-11-14
comments: true
categories: OAuth
---

While OpenID and WS-Federation focus on delegating user identity (or a
collection of identity claims), OAuth was designed to address a
different and complementary scenario, the delegation of user
authorization. In few words, OAuth allows a client application to obtain
user consent (as access tokens) for executing operations over private
resources on his behalf.

The analogy given by Eran Hammer Lahav in this post "[Explaining
OAuth](http://www.hueniverse.com/hueniverse/2007/09/explaining-oaut.html)"
is very close to what the specification tries to address,

*"Many luxury cars today come with a valet key. It is a special key you
give the parking attendant and unlike your regular key, will not allow
the car to drive more than a mile or two. Some valet keys will not open
the trunk, while others will block access to your onboard cell phone
address book. Regardless of what restrictions the valet key imposes, the
idea is very clever. You give someone limited access to your car with a
special key, while using another key to unlock everything else."*

Now, if we analyze the specification in more detail, we will see that
the real purpose behind OAuth is to create a network of collaboration 
between applications. It will not be necessary anymore to keep all our
stuff just in a single place, we can have for instance our pictures in a
website, our contacts in another place and a third application making
use of them, all these applications collaborating together.

If you want to know more about how OAuth works, you should read the
following posts

-   ["Begginer's Guide to OAuth - Part
    I"](http://www.hueniverse.com/hueniverse/2007/10/beginners-guide.html)
-   ["Begginer's Guide to OAuth - Part II - Protocol
    Workflow"](http://www.hueniverse.com/hueniverse/2007/10/beginners-gui-1.html)
-   ["Begginer's Guide to OAuth - Part III - Security
    Architecture"](http://www.hueniverse.com/hueniverse/2008/10/beginners-guide.html)

When I initially said that OpenID and OAuth complement each other, I
meant that the user can first authenticated by an OpenID provider, and
then redirected to the relying party to obtain his consent.
(Authorization for consuming a private resource).

[Alex Henderson (Aka Bittercoder)](http://blog.bittercoder.com/) has
written a pretty good OAuth library in .NET for implementing an OAuth
consumer and service provider. The library is available
[here](http://code.google.com/p/devdefined-tools/wiki/OAuth) under a MIT
license (do wherever you want with it), and it is very easy to use. Alex
has definitively made a very good work.

My WCF channel implementation for OAuth mounts on top of his library and
it basically transforms a OAuth token into a .NET security principal
that can be used later within the service implementation. The channel is
implemented as a RequestInterceptor, one of new features introduced in
the REST WCF Starter Kit. This interceptor basically captures the
request at channel level and performs all the validations required by
OAuth. The following sample illustrates how the interceptors can be
plugged into an existing service host (service.svc),

\<%@ ServiceHost Language="C\#" Debug="true"
Service="ExampleOAuthChannel.FeedService"
Factory="ExampleOAuthChannel.AppServiceHostFactory"%\>

using System;\
using System.ServiceModel;\
using System.ServiceModel.Activation;\
using Microsoft.ServiceModel.Web;

namespace ExampleOAuthChannel \
{\
  class AppServiceHostFactory : ServiceHostFactory\
  {\
    protected override ServiceHost CreateServiceHost(Type serviceType,
Uri[] baseAddresses)\
    {\
        WebServiceHost2 result = new WebServiceHost2(serviceType, true,
baseAddresses);\
        result.Interceptors.Add(new OAuthChannel.OAuthInterceptor(\
   OAuthServicesLocator.Provider,
OAuthServicesLocator.AccessTokenRepository));\
        return result;\
    }\
  }\
}

OAuthServicesLocator.Provider and
OAuthServiceLocator.AccessTokenRepository are just part of the OAuth
implementation.

The interaction (and all the messages interchanged) between the consumer
and the provider was very well summarized by Alex in this post ["OAuth
for
beginners"](http://blog.bittercoder.com/PermaLink,guid,83488336-290d-4c4b-a314-14fe255e5b4e.aspx)

The following code illustrates some of the functionality implemented in
the OAuth interceptor,

Message request = requestContext.RequestMessage;

HttpRequestMessageProperty requestProperty =
(HttpRequestMessageProperty)request.Properties[HttpRequestMessageProperty.Name];

 

OAuthContext context = new
OAuthContextBuilder().FromUri(requestProperty.Method,
request.Headers.To);

 

try

{

    \_provider.AccessProtectedResourceRequest(context);

 

    OAuthChannel.Models.AccessToken accessToken =
\_repository.GetToken(context.Token);

 

    TokenPrincipal principal = new TokenPrincipal(

        new GenericIdentity(accessToken.UserName, "OAuth"),

        accessToken.Roles,

        accessToken);

 

    InitializeSecurityContext(request, principal);

}

catch (OAuthException authEx)

{

    XElement response = XElement.Load(new StringReader("\<?xml
version=\\"1.0\\" encoding=\\"utf-8\\"?\>\<html
xmlns=\\"http://www.w3.org/1999/xhtml\\" version=\\"-//W3C//DTD XHTML
2.0//EN\\" xml:lang=\\"en\\"
xmlns:xsi=\\"http://www.w3.org/2001/XMLSchema-instance\\"
xsi:schemaLocation=\\"http://www.w3.org/1999/xhtml
http://www.w3.org/MarkUp/SCHEMA/xhtml2.xsd\\"\>\<HEAD\>\<TITLE\>Request
Error\</TITLE\>\</HEAD\>\<BODY\>\<DIV id=\\"content\\"\>\<P
class=\\"heading1\\"\>\<B\>" +
HttpUtility.HtmlEncode(authEx.Report.ToString()) +
"\</B\>\</P\>\</DIV\>\</BODY\>\</html\>"));

    Message reply = Message.CreateMessage(MessageVersion.None, null,
response);

    HttpResponseMessageProperty responseProperty = new
HttpResponseMessageProperty() { StatusCode = HttpStatusCode.Forbidden,
StatusDescription = authEx.Report.ToString() };

    responseProperty.Headers[HttpResponseHeader.ContentType] =
"text/html";

    reply.Properties[HttpResponseMessageProperty.Name] =
responseProperty;

    requestContext.Reply(reply);

 

    requestContext = null;

}

It basically validates the OAuth ticket using the library written by
Alex and initializes a new principal containing the ticket identity. If
the ticket can not be validated for some reason, it returns a friendly
exception to the consumer.

UPDATE: Alex has now include the channel as part of the OAuth Library.
It is available under the following links,

[http://devdefined-tools.googlecode.com/svn/trunk/projects/oauth/DevDefined.OAuth.Wcf/](http://devdefined-tools.googlecode.com/svn/trunk/projects/oauth/DevDefined.OAuth.Wcf/)

[http://devdefined-tools.googlecode.com/svn/trunk/projects/oauth/ExampleOAuthChannel/](http://devdefined-tools.googlecode.com/svn/trunk/projects/oauth/ExampleOAuthChannel/)

Coming next  "Using the WCF OAuth channel with an ADO.NET service" (The
complete source code will be available as part of that post)

