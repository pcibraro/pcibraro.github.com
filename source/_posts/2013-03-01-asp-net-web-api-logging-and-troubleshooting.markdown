---
layout: post
title: "ASP.NET Web API Logging and Troubleshooting"
date: 2013-03-01
comments: true
categories: .NET
---

ASP.NET ships with two built-in mechanisms for doing logging and
troubleshooting.  Chasing errors without knowing these two mechanisms
might be a daunting task, specially if they happen in the runtime
pipeline much before a message gets to a handler or a controller.

The first mechanism is the error policy. You can configure the error
policy preferences as part of the configuration object
(HttpConfiguration) in the IncludeErrorDetailPolicy property. This is
just an enum that instructs Web API about how to deal with exceptions.

The possible values for this enum are,

-   Default: It’s uses the customErrors configuration settings if you
    are using ASP.NET as host or LocalOnly for self-host.
-   LocalOnly: Only includes error details for local requests
-   Always: Always includes error details
-   Never: Never includes error details

When an exception happens, Web API will check the value on this setting
for including details about the exception in the response message or
not. For example, if Always is enabled, Web API will serialize the
exception details as part of the message that you get as response.

The second mechanism is Tracing. Tracing is a service that you can
inject as part of the configuration object as well. The default
implementation does do anything.

```csharp
public static void Register(HttpConfiguration config)
{
    config.Services.Replace(typeof(ITraceWriter), new MyTracer());
}
```

MyTracer is a custom implementation of the ITraceWriter service, which
Web API uses for tracing purposes. This is a general tracing mechanism,
so Web API will call it for logging everything and not just errors.

```csharp
public class MyTracer : ITraceWriter
{
    public void Trace(HttpRequestMessage request, string category, TraceLevel level, 
        Action<TraceRecord> traceAction)
    {
        TraceRecord rec = new TraceRecord(request, category, level);
        traceAction(rec);
        WriteTrace(rec);
    }

    protected void WriteTrace(TraceRecord rec)
    {
        var message = string.Format("{0};{1};{2}", 
            rec.Operator, rec.Operation, rec.Message);
        System.Diagnostics.Trace.WriteLine(message, rec.Category);
    }
}
```

If any of these two work for you, you can still use an Error Filter. 
Tugberk has written a blog post about how to integrate ELMAH with an
Error Filter in Web API
[here](http://www.tugberkugurlu.com/archive/asp-net-web-api-and-elmah-integration).

