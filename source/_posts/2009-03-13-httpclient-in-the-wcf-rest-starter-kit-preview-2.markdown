---
layout: post
title: "HttpClient in the WCF REST Starter Kit Preview 2"
date: 2009-03-13
comments: true
categories: REST
---

HttpClient is a new utility class introduced in the WCF REST Startert
Kit Preview 2 for consuming REST services. This new class is made up of
three different parts,

​1. A rich object model to manipulate the Http request and response
objects in a more natural way. This is done through the use of the
HttpRequestMessage and HttpResponseMessage classes.

​2. Some overloads or extension methods to reduce the number of code
lines required to consume a service. For example, the HttpClient class
itself contains method overloads to send messages to a service through
some of the well-know http verbs, such as Get, Post, Put, Delete, Head
or Options.  The HttpRequestMessage and HttpResponse classes also
contain methods for serializing/deserializing the incoming/outgoing
messages in different formats.

HttpClient client = new HttpClient();           

var echo = client.Get(new
Uri("http://localhost:1449/MyRestService/Service.svc/Echo")).Content.ReadAsString();

The sample code above sends a Http Get message to an “Echo” service and
gets the response as a simple string.

​3. An extensible execution pipeline made up of customizable steps or
stages. This pipeline takes the form of a pipeline controller pattern
where each step executes one after another to return back the execution
control to the pipeline (It is basically a coordinator). This feature
gives enough flexibility to perform additional work over the
request/response messages such as validations or caching to name a few.

Every step in the pipeline is represented by a HttpStage class. This
class contains two overloads,

public class MyCustomStage : HttpStage

{

    protected internal override void
ProcessRequestAndTryGetResponse(HttpRequestMessage request, out
HttpResponseMessage response, out object state)

    {

    }

    protected internal override void ProcessResponse(HttpResponseMessage
response, object state)

    {

    }

}

The “ProcessRequestAndTryGetResponse” is executed first in the pipeline,
and it allows doing some processing over the request message. It is also
possible in this method to return a response message, if that happens,
all the stages that come after this one are not executed and the
response is returned to the client application. As result of this, the
REST service does not get called because the last stage in the pipeline
is usually the one that sends the request through http to the service.
This approach is generally useful for performing some caching or mocking
the responses from unit tests.

The following example interrupts the pipeline execution and returns an
string (“myfoo”) as response message,

public class MyStage : HttpStage

{

    protected internal override void
ProcessRequestAndTryGetResponse(HttpRequestMessage request, out
HttpResponseMessage response, out object state)

    {

        response = new HttpResponseMessage

        {

            Content = HttpContent.Create("myfoo"),

            Method = "GET",

            StatusCode = System.Net.HttpStatusCode.OK,

            Uri = request.Uri

        };

        state = null;

    }

    protected internal override void ProcessResponse(HttpResponseMessage
response, object state)

    {

    }

}

The “ProcessResponse” method is executed after a response message is
found in the pipeline. As it name states, it allows doing some
additional processing over that response message.

A more specialized version of these stages is also provided by the
starter kit, HttpProcessingStage

public class MyCustomStage : HttpProcessingStage

{

    public override void ProcessRequest(HttpRequestMessage request)

    {

    }

    public override void ProcessResponse(HttpResponseMessage response)

    {

    }

}

This class derives from the standard “HttpStage” and adds some plumbing
code to simplify the work of a developer only interested in inspecting
or modifying the request or response messages. It works like a Message
Inspector in WCF.

These custom stages can be added to the HttpClient trough the “Stages”
collection,

client.Stages.Add(new MyStage());

If you want to start playing with this new useful class, go an grab the
latest preview of the [Starter kit from
Codeplex](http://www.codeplex.com/aspnet/Wiki/View.aspx?title=WCF%20REST&referringTitle=Home).

