---
layout: post
title: "Streaming large content from a WCF RESTFul service"
date: 2009-03-27
comments: true
categories: REST
---

Streaming large content such as media content, images or files is a
common scenario for RESTful services. If a scenario like this is not
well addressed or implemented on the service side, there is a high risk
of consuming server resources like memory or CPU in a matter of seconds.
This usually happens when the complete content is loaded in memory
before it is transferred to the service consumer.

The WCF REST Starter kit introduced a new mechanism for addressing this
scenario, an “AdapterStream” utility class, which basically pushes small
pieces of content (and thus, only small memory buffers are used) to the
client application as it becomes necessary.

The kit also comes with an example “PushStyleStreaming” that shows this
class in action

Stream GetImage(string text)

{

    if (string.IsNullOrEmpty(text))

    {

        throw new WebProtocolException(HttpStatusCode.BadRequest, "text
must be specified", null);

    }

    Bitmap theBitmap = GenerateImage(text);

    WebOperationContext.Current.OutgoingResponse.ContentType =
"image/jpeg";

    return new AdapterStream((stream) =\> theBitmap.Save(stream,
ImageFormat.Jpeg));

}

In the code above, the AdapterStream is used to transfer an image to a
service consumer. As you can also notice in that code, that class
receives a lambda  expression or Action\<T\> delegate in the
constructor. That provides some flexibility at the moment of generating
the final stream that will be sent to the client application.

Another example that uses an TextWriter for generating text content,

  new AdapterStream((writer) =\>

    {

        writer.WriteLine("You said: ");

        writer.WriteLine(text);

        writer.WriteLine("Didn't you?");

        writer.Flush();

    }, Encoding.UTF8);

In addition to this feature, you might also want to have a better
control of service usage by restricting the [throttling
settings](http://kennyw.com/indigo/150). This is a good thing about REST
services implemented with the WCF stack. Other implementations such as
the ASP.NET MVC rely on the ASP.NET for handling this aspect, where
these settings are applied only at application domain level (For all
services running in the same app).

