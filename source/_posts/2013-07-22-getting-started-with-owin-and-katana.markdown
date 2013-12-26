---
layout: post
title: "Getting started with Owin and Katana"
date: 2013-07-22
comments: true
categories: .NET
---

Introduction
------------

The .NET ecosystem offers today a lot of alternatives for developing web
applications. You can either use any of the frameworks supported by
Microsoft with ASP.NET such as Forms, MVC or Web API, or any other open
source alternative like FubuMVC, ServiceStack, NancyFx or OpenRasta to
name a few. From an architecture standpoint, all these frameworks have
three main layers in common (in spite of the difference with the
implementation details), hosting, middleware, and application.

The hosting layer is responsible for managing the underline process,
where the http connections are established and managed, and also to
materialize those connections into request/response objects that are
sent to the upper layers. For example, the hosting layer could be
ASP.NET running in IIS, or it could also be a console application that
uses the http listener directly.

The middleware layer provides common infrastructure services, which runs
at low level in the http stack, such as security, caching, or any other
concern that can be handled at this level.

The application layer is where the applications or services are
implemented using the features available in the framework.

However, the distinction between the middleware and application layers
is not always clear, which causes that many of the middleware services
end up being implemented in the application layer. For example, you
could implement middleware services for security in ASP.NET MVC using
filters, which rely on features specific to the application level in
that framework. This make almost impossible to reuse all these services
across all the available frameworks, so you will find different
implementations of the same services for the different frameworks. To
give an example, basic authentication is something that could be
implemented as middleware service and reused, but today is implemented
differently in each framework.

Someone would suggest that a ASP.NET Http Module could be a good option
for implementing a middleware service, but that’s not necessarily true
as it would only work when the hosting layer uses ASP.NET.

This exact problem is what [OWIN
specification](http://owin.org/spec/owin-1.0.0.html) tries to address by
providing a common abstraction for any http-aware service or application
with minimal dependencies on existing frameworks or implementations.  At
very core level, OWIN defines a handler, which is represented as an
application delegate,

using AppFunc = Func\< \
        IDictionary\<string, object\>, // Environment \
        Task\>; // Done

The idea is to represent the three layers discussed above as a set of
handlers using the “chain of responsibility” pattern. The environment
argument is a dictionary that the different handlers use to exchange
values, and the next handler in the chain is represented with the task
argument. Having said that, the first handler in the chain would be
responsible for the hosting layer, and it should populate the
environment with keys that describe the received http request message,
such as scheme, port, request path, headers, etc. All those common
headers are discussed as part of the specification.

Katana is a Microsoft implementation of the OWIN specification, which
includes handlers for the hosting and middleware layers. It’s an open
source project, which can be found in this
[location](http://katanaproject.codeplex.com/SourceControl/latest). All
the investment that Microsoft has done so far for ASP.NET in the
different application frameworks (i.e. MVC or Web API), it’s now being
refactored as OWIN handlers that can be reused across different
application frameworks. For example, cookie authentication or even OAuth
are being implemented as middleware services that can be reused not only
by ASP.NET web apps but also for other application frameworks in the
open source world.

The application layer is also implemented as another OWIN handler, which
is responsible for setting up all the infrastructure required to handle
the request/response at the application level. You will find application
handlers for ASP.NET MVC, ASP.NET Web API, SignalR, or NancyFx to give
an example. 

Gettting Started with OWIN and Katana
-------------------------------------

The easiest way to get started with OWIN is to use the assembly
distributed as nuget package. You will find a single nuget package for
this called “OWIN”. This assembly only contains an interface
IAppBuilder, which can be used to configure all the handlers that you
want to use in your web application. This is a very low level interface
that you won’t have to implement yourself if you use Katana.

In addition to the “OWIN” assembly, you will find two additional nuget
packages

-   Owin.Extensions: Provides a set of very useful extensions methods
    for the IAppBuilder interface that you can use to configure the
    handlers.
-   Owin.Types: Provides a set of wrappers to represent the environment
    dictionary as a typed object representing the request and response
    messages.

You can implement your first OWIN handler using these three packages
(You will not able to run it without the hosting handlers provided by
Katana).

```csharp
public class MyMiddleware
{
  Func<IDictionary<string, object>, Task> next;

  public MyMiddleware(Func<IDictionary<string, object>, Task> next)
  {
    this.next = next;
  }

  public Task Invoke(IDictionary<string, object> env)
  {
    var headers = (IDictionary<string, string[]>)env["owin.RequestHeaders"];

    var url = string.Format("{0}://{1}{2}{3}",
                (string)env["owin.RequestScheme"],
                headers["host"].First(),
                (string)env["owin.RequestPathBase"],
                (string)env["owin.RequestPath"]);

    if ((string)env["owin.RequestQueryString"] != "")
      url += "?" + (string)env["owin.RequestQueryString"];

    Console.WriteLine("The received URL is {0}", url);

    return this.next(env);
  }
}
```

This is a low level handler that sends the request url to the console.
As you can see, it relies on some existing properties in the
environment, which should have been set by Katana in the hosting layer.
The handler also receives the next handler to call in the chain as part
of the constructor.

The entry point in any Katana application must be a class that receives
the IAppBuilder interface as argument to configure all the handlers, as
it is shown below,

```csharp
 public class Startup
 {
    public void Configuration(IAppBuilder app)
    {
       app.UseType<MyMiddleware>();       
    }
 }
```

UseType is one the extension methods provided by “Owin.Extensions” to
configure a new handler in the pipeline. You can also other extension
methods to define a handler on the fly using delegates as this one,

```csharp
app.UseHandlerAsync(async (req, res) =>
{
    res.ContentType = "text/plain";
    await res.WriteAsync("Hello World");
});
```

There is one more nuget package that you will find useful to create OWIN
handlers, which is “Microsoft.Owin”. This is a package provided as part
of Katana (It’s still a pre-release), which extends the Owin Request and
Response classes with more properties and methods, and also helper
methods to deal with these classes.

Hosting an application with Katana
----------------------------------

Katana provides three options to host a web application.

​1. OwinHost.exe. It’s a command tool that you can use to run the
application. You can use multiple command arguments to specify the
assembly with the entry point for configuring the IAppBuilder or other
configuration settings such as the port number or tracing. This is tool
is available as part of the nuget package “OwinHost”.

​2. .NET application. There is a WebApp class that you can use to
self-host a web application in any regular .NET application such as
console app or a windows service. This class is available as part of the
“Microsoft.Owin.Hosting” assembly.

```csharp
using (WebApp.Start<Startup>(new StartOptions { Port = 5000 }))
{
   Console.WriteLine("Press Enter to quit.");
   Console.ReadKey();
}
```

You need to pass the startup entry as argument to the static method
“Start”.

​3. ASP.NET. You need to reference the nuget package
“Microsoft.Owin.Hosting.SystemWeb” (Pre-Release) from your web
application project, and include the Startup class in that project as
well.

