---
layout: post
title: "ASP.NET MVC, WCF REST and Data Services – When to use what for RESTful services"
date: 2010-10-08
comments: true
categories: .NET
---

*Disclaimer: This post only contains personal opinions about this
subject*

In the way I see it, REST mainly comes in for two scenarios, when you
need to expose some AJAX endpoints for your web application or when you
need to expose an API to external applications through well defined
services.  

For AJAX endpoints, the scenario is clear, you want to expose some data
for being consumed by different web pages with client scripts. At this
point, what you expose is either HTML, text, plain XML, or JSON, which
is the most common, compact and efficient format for dealing with data
in javascript. Different client libraries like JQuery or MooTools
already come with built-in support for JSON. This data is not intended
for being consumed by other clients rather than the pages running in the
same web domain. In fact, some companies add custom logic in these
endpoints for responding only to requests originated in the same domain.

For RESTful services, the scenario is completely different, you want to
expose data or business logic to many different client applications
through a well know API. Good examples of REST services are twitter, the
Windows Azure storage services or the Amazon S3 services to name a few.

Smart client applications implemented with Silverlight, Flex or Adobe
Air also represent client applications that can be included in this
category with some restrictions, as they can not make cross domain call
to http services by default unless you override the cross-domain
policies.   

A common misconception is to think that REST services only mean CRUD for
data, which is one of the most common scenarios, but business workflows
can also be exposed through this kind of service as it is shown in this
article [“How to get a cup of
coffe”](http://www.infoq.com/articles/webber-rest-workflow)

When it comes to the .NET world, you have three options for implementing
REST services (I am not considering third party framework or OS projects
in this post),

1.  ASP.NET MVC
2.  WCF REST
3.  WCF Data Services (OData)

ASP.NET MVC
-----------

This framework represents the implementation of the popular
model-view-controller (MVC) pattern for building web applications in the
.NET space. The reason for moving traditional web application
development toward this pattern is to produce more testable components
by having a clean separation of concerns. The three components in this
architecture are the model, which only represents data that is shared
between the view and the controller, the view, which knows how to render
output results for different web agents (i.e. a web browser), and
finally the controller, which coordinates the execution of an use case
and the place where all the main logic associated to the application
lives on. Testing ASP.NET Forms applications was pretty hard, as the
implementation of a page usually mixed business logic with rendering
details, the view and the controller were tied together, and therefore
you did not have a way to focus testing efforts on the business logic
only. You could prepare your asp.net forms application to use MVC or MVP
(Model-View-Presenter) patterns to have all that logic separated, but
the framework itself did not enforce that.

On the other hand, in ASP.NET MVC, the framework itself drives the
design of the web application to follow an MVC pattern. Although it is
not common, developers can still make terrible mistakes by putting
business logic in the views, but in general, the main logic for the
application will be in the controllers, and those are the ones you will
want to test.

In addition, controllers are TDD friendly, and the ASP.NET MVC team has
made a great work by making sure that all the ASP.NET intrinsic objects
like the Http Context, Sessions, Request or Response can be mocked in an
unit test.

While this framework represents a great addition for building web
applications on top of ASP.NET, the API or some components in the
framework (like view engines) not necessarily make a lot of sense when
building stand alone services. I am not saying that you could not build
REST services with ASP.NET MVC, but you will have to leverage some of
the framework extensibility points to support some of the scenarios you
might want to use with this kind of services.

Content negotiation is a good example of an scenario not supported by
default in the framework, but something you can implement on your own
using some of the available extensions. For example, if you do not want
to tie your controller method implementation to an specific content
type, you should return an specific ActionResult implementation that
would know how to handle and serialize the response in all the supported
content types. Same thing for request messages, you might need to
implement model binders for mapping different content types to the
objects actually expected by the controller.

The public API for the controller also exposes methods that are more
oriented to build web applications rather than services. For instance,
you have methods for rendering javascript content, or for storing
temporary data for the views.

If you are already developing a website with ASP.NET MVC, and you need
to expose some AJAX endpoints (which actually represents UI driven
services) for being consumed in the views, probably the best thing you
can do is to implement them with MVC too as operations in the
controller. It does not make sense at this point to bring WCF to
implement those, as it would only complicate the overall architecture of
the application. WCF would only make sense if you need to implement some
business driven services that need to be consumed by your MVC
application and some other possible clients applications as well.

This framework also introduced a new routing mechanism for mapping URLs
to controller actions in ASP.NET, making possible to have friendly and
nice URLS for exposing the different resources in the API.

As this framework is layered on top of ASP.NET, one thing you might find
complicated to implement is security. Security in ASP.NET is commonly
tied to a web application, so it is hard to support schemes where you
need different authentication mechanisms for your services. For example,
if you have basic authentication enabled for the web application hosting
the services, it would be complicated to support other authentication
mechanism like OAuth. You can  develop custom modules for handling these
scenarios, but that is something else you need to implement.   

WCF REST
--------

The WCF Web Http programming model was first introduced as part of the
.NET framework 3.5 SP1 for building non-SOAP http services that might
follow or not the different REST architectural constraint. This new
model brought to the scene some new capabilities for WCF by adding new
attributes in the service model ([WebGet] and [WebInvoke]) for routing
messages to service operations through URIs and Http methods, behaviors
for doing same basic content negotiation and exception handling, a new
WebOperationContext static object for getting access and controlling
different http header and messages, and finally a new binding
WebHttpBinding for handling some underline details related to the http
protocol.

The WCF team later released an REST starter kit in codeplex with new
features on top of this web programming model to help developers to
produce more RESTful services. This starter kit also included a
combination of examples and Visual Studio templates for showing how some
of the REST constraints could be implemented in WCF, and also a couple
of interesting features to support help pages, output caching,
intercepting request messages and a very useful Http client library for
consuming existing services (HttpClient)

Many of those features were finally included in the latest WCF release,
4.0, and also the ability of routing messages to the services with
friendly URLs using the same ASP.NET routing mechanism that ASP.NET MVC
uses.

As services are first citizens in WCF, you have exclusive control over
security, message size quotas and throttling for every individual
services and not for all services running in a host as it would happen
with ASP.NET.  In addition, WCF provides its own hosting infrastructure,
which is not dependant of ASP.NET so it is possible to self hosting
services in any regular .NET application like a windows service for
example.

In the case of hosting services in ASP.NET with IIS, previous versions
of WCF (3.0 and 3.5) relied on a file with “svc” extension to activate
the service host when a new message arrived. WCF 4.0 now supports
file-less activation for services hosted in ASP.NET, which relies on a
configuration setting, and also a mechanism based on Http routing
equivalent to what ASP.NET MVC provides, making possible to support
friendly URLs. However, there is an slight difference in the way this
last one works compared to ASP.NET MVC. In ASP.NET MVC, a route
specifies the controller and also the operation or method that should
handle a message. In WCF, the route is associated to a factory that
knows how to create new instances of the service host associated to the
service, and URI templates attached to [WebGet] and [WebInkoke]
attributes in the operations take care of the final routing. This
approach works much better in the way I see it, as you can create an URI
schema more oriented to resources, and route messages based on Http
Verbs as well without needing to redefine additional routes. 

The support for doing TDD at this point is somehow limited fore the fact
that services rely on the static context class for getting and setting
http headers, making very difficult to initialize that one in a test or
mock it for setting some expectations.

The content negotiation story was improved in WCF 4.0, but it still
needs some twists to make it complete as you might need to extend the
default WebContentTypeMapper class for supporting custom media types
other than the standard “application/xml” for xml and “application/json”
for JSON.

The WCF team is seriously considering to improve these last two aspects
and adding some other capabilities to the stack for a next version.

WCF Data Services
-----------------

WCF Data Services, formerly called ADO.NET Data Services, was introduced
in the .NET stack as way of making any IQueryable data source public to
the world through a REST API. Although a WCF Data Service sits on top of
the WCF Web programming model, and therefore is a regular WCF service, I
wanted to make a distinction here for the fact that this kind of service
exposes metadata for the consumers, and also adds some restrictions for
the URIs and the types that can be exposed in the service. All these
features and restrictions have been documented and published as a
separate specification known as OData. 

The framework includes a set of providers or extensibility points that
you can customize to make your model writable, extend the available
metadata, intercepting messages or supporting different paging and
querying schemas. 

A WCF Data Service basically uses URI segments as mechanism for
expressing queries that can be translated to an underline linq provider,
making possible to execute the queries on the data source itself, and
not something that happens in memory. The result of executing those
queries is what finally got returned as response from the service.
Therefore, WCF Data services use URI segments to express many of
supported linq operators, and the entities that need to be retrieved.
This capability of course is what limit the URI space that you can use
on your service, as any URI that does not follow the OData standard will
result in an error.

Content negotiation is also limited to two media types, JSON and Xml
Atom, and the content payload itself is restricted to specific types
that you can find as part of the OData specification.

Besides of those two limitations, WCF Data Service is still extremely
useful for exposing a complete data set with query capabilities through
a REST interface with just a few lines of code. JSON and Atom are two
very accepted formats nowadays, making this technology very appealing
for exposing data that can easily be consumed by any existing client
platform, and even web browsers.

Also, for Web applications with ajax and smart client applications, you
do not need to reinvent the wheel and create a complete set of services
for just exposing data with a CRUD interface. You get your existing data
model, configure some views or filters for the data you want to expose
in the model in the data service itself, and that is all.          

