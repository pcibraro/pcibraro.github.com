---
layout: post
title: "Developing RESTful services with JSON and POX support in the ASP.NET MVC"
date: 2009-04-17
comments: true
categories: ASP.NET
---

Many of the features available out of the box today in the ASP.NET MVC
framework are only intended to develop web applications using REST
principles. There is not support for accepting incoming messages encoded
as JSON or plain old XML (POX), or even support for returning POX from a
controller action. Only form-urlencoded data is accepted by default for
incoming messages, and JSON/HTML (with support of the view engines) for
outgoing messages in controller actions.

However, thanks to the different extensibility points that the MVC
provides for hooking up custom code, supporting different scenarios for
RESTful services tend to be something really easy to implement.

In this framework, we basically have two extensibility points that
deserve some attention, Action Filters and Action Results.

Action filters intercept messages before they are dispatched to the
controller action (or operation), and just after the operation returns a
result. They are basically equivalent to the Dispatch Message
Interceptors in WCF, or the new Message Interceptors in the WCF REST
Starter kit (although they work more at a deeper level in the WCF
processing pipeline).

Action results know how to serialize and write an object (the controller
action result) into the response stream. Since they also have access to
the Response object, additional work can be done there to manipulate
some response settings or add output headers.

The framework also support Model Binders, which basically knows how to
create an object representing the model expected by the controller
action from some scalar values. Those scalar values are parsed from the
incoming message (encoded as form-urlencoded), and then passed to the
binder. However, they do not seem to add any value for implementing
RESTful services with support for JSON or XML.

Omar Al Zabir has already written [an nice
post](http://weblogs.asp.net/omarzabir/archive/2008/10/03/create-rest-api-using-asp-net-mvc-that-speaks-both-json-and-plain-xml.aspx)
on how to implement RESTful services with the ASP.NET MVC that speak
JSON and XML using action filters and action results.

ATOM and other syndication formats can also be handled as XML. For this,
the Web Programming Model in WCF comes with a couple of classes,
Rss20FeedFormatter and Atom10FeedFormator, which are Data Contracts and
also IXmlSerializable classes. Therefore, if your operation returns an
instance of any of these classes, the filters created by Omar would
address this scenario as well. Regarding Conditional get support, you
will have to implement it in the action itself or a custom filter. (You
will have to do the same thing if you decide to go with the WCF Web
Programming Model).

Caching is another feature that you might want to use at the moment of
developing RESTful services. Fortunately, the MVC also comes with an
special action filter “OutputCache” that was built on top the ASP.NET
cache, so it provides the same caching capabilities that you may use for
normal ASP.NET pages.

