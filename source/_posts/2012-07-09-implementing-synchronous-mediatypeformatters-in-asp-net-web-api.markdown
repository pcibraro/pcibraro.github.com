---
layout: post
title: "Implementing synchronous MediaTypeFormatters in ASP.NET Web API"
date: 2012-07-09
comments: true
categories: .NET
---

One of main characteristics of MediaTypeFormatter’s in ASP.NET Web API
is that they leverage the Task Parallel Library (TPL) for reading or
writing an model into an stream. When you derive your class from the
base class MediaTypeFormatter, you have to either implement the
WriteToStreamAsync or ReadFromStreamAsync methods for writing or reading
a model from a stream respectively.

These two methods return a Task, which internally does all the
serialization work, as it is illustrated bellow.

~~~~ {.alt}
public abstract class MediaTypeFormatter
~~~~

    {

~~~~ {.alt}
  public virtual Task WriteToStreamAsync(Type type, object value, 
~~~~

         Stream writeStream, HttpContent content, TransportContext transportContext);

~~~~ {.alt}
  public virtual Task<object> ReadFromStreamAsync(Type type, Stream readStream, 
~~~~

         HttpContent content, IFormatterLogger formatterLogger);

~~~~ {.alt}
}
~~~~

     

However, most of the times, serialization is a safe operation that can
be done synchronously. In fact, many of the serializer classes you will
find in the .NET framework only provide sync methods. So the question
is, how you can transform that synchronous work into a Task ?.

Creating a new task using the method Task.Factory.StartNew for doing all
the serialization work would be probably the typical answer. That would
work, as a new task is going to be scheduled. However, that might
involve some unnecessary context switches, which are out of our control
and might be affect performance on server code specially.  

If you take a look at the source code of the MediaTypeFormatters shipped
as part of the framework, you will notice that they actually using
another pattern, which uses a TaskCompletionSource class.

~~~~ {.alt}
public Task WriteToStreamAsync(Type type, object value, Stream writeStream, 
~~~~

    HttpContent content, TransportContext transportContext)

~~~~ {.alt}
{
~~~~

     

~~~~ {.alt}
  var tsc = new TaskCompletionSource<AsyncVoid>();
~~~~

      tsc.SetResult(default(AsyncVoid));

~~~~ {.alt}
 
~~~~

      //Do all the serialization work here synchronously

~~~~ {.alt}
 
~~~~

      return tsc.Task;

~~~~ {.alt}
}
~~~~

     

~~~~ {.alt}
/// <summary>
~~~~

    /// Used as the T in a "conversion" of a Task into a Task{T}

~~~~ {.alt}
/// </summary>
~~~~

    private struct AsyncVoid

~~~~ {.alt}
{
~~~~

    }

They are basically doing all the serialization work synchronously and
using a TaskCompletionSource for returning a task already done.

To conclude this post, this is another approach you might want to
consider when using serializers that are not compatible with an async
model.

**Update:** Henrik Nielsen from the ASP.NET team pointed out the
existence of a built-in media type formatter for writing sync
formatters. BufferedMediaTypeFormatter
[http://t.co/FxOfeI5x](http://t.co/FxOfeI5x)

