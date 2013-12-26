---
layout: post
title: "Adding documentation to WCF Restful services with the REST Starter Kit"
date: 2008-11-04
comments: true
categories: REST
---

Automatic documentation is another cool feature introduced in the WCF
REST Starter kit. While documentation is an important aspect in the
development process, unfortunately there is not an standard mode or
guidance yet about how this should be done for REST services. This new
feature comes to help a little in this aspect of the process.

As part of the starter kit, the WCF team has created a new [WebHelp]
attribute that can be used to annotate existing web operations with
human-readable descriptions. The [WebHelp] attribute (as the WebCache
attribute) is an operation behavior implementation that receives a
simple string to describe the operation behavior. The operation
arguments are automatically reflected and shown as part of the service
documentation.

[WebInvoke(UriTemplate = "DoWork?json", ResponseFormat =
WebMessageFormat.Json)]

[WebHelp(Comment="This method returns a HTTP status code Conflict with a
custom json error body")]

[OperationContract]

 SampleResponseBody DoWorkJson(SampleRequestBody request)

Now, the question is, once we have this attribute applied to all the web
operations of an existing service, where is the documentation actually
published ?. Well, this feature also requires the use of a new service
host "WebServiceHost2" that comes with the starter kit, which among
other things publishes a new endpoint "/help" for the service.
Therefore, the final documentation for a service will be available at
"ServiceUri" + "/help". For example,
[http://localhost/MyService.svc/help](http://localhost/MyService.svc/help)

![](/images/legacy/ServiceHelpSmall.jpg)

 

As you can see in the image above, an Atom feed is created and published
in the /help endpoint. Each entry in the feed provides some
documentation about any of the available web operations.

The new service host can be configured in a existing service modifying
the .svc file (If the service runs on IIS),

\<%@ ServiceHost Language="C\#" Debug="true" Service="Service.Service"
Factory="Microsoft.ServiceModel.Web.WebServiceHost2Factory"%\>

