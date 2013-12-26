---
layout: post
title: "OpenRasta, an open source alternative for developing Restful services"
date: 2009-04-22
comments: true
categories: .NET
---

[OpenRasta](http://serialseb.blogspot.com/2008/12/openrasta-is-available.html)
(OR) is the name of a new open source framework for developing Restful
applications (Web applications or services) created by [Sebastien
Lambla](http://serialseb.blogspot.com/). As the ASP.NET MVC, this
framework is also an implementation of the MVC pattern, which focus
mainly on a good separation of concerns, and provide different
extensibility points where the developer can plugging custom code at the
moment of developing applications.

The framework is now on Beta 2 stage, Sebastien and other guys are
currently working on improving the hosting infrastructure and public
API. They are, of course, very open at this point to receive feedback or
suggestions for new features, or improvements to existing ones. If you
have some interest in participating of this project, you can start
joining the [mailing list](http://groups.google.com/group/openrasta/).

In order to understand how OR works under the hood, I will start by
describing some basic concepts that a developer should know to implement
an application from scratch with this framework.

**Handlers**

A “handler” is one of the main building blocks in OR. It represents the
implementation of the service itself, and it is generally mapped to
single resource through an URI. Therefore, any access to an URI will get
resolved by OR to an specific handler.

From an implementation point of view, a handler is simple class that
expose methods for handling different http verbs. By convention, a
method should be called as the Http verb that supports, for example, Get
or Post. However, nothing prevent you from giving more friendly names to
the methods (or handler operations), there is an special custom
attribute “HttpOperationAttribute” that you can use to associate the
method with an http verb.

public class CustomerHandler

{

    public Customer Get(int customerId)

    {

        return new Customer { FirstName = "foo", LastName = "bar" };

    }

    public OperationResult Put(int id, Customer customer)

    {

        return new OperationResult.Modified { ResponseResource =
customer };

    }

    [HttpOperation(HttpMethod.POST)]

    public OperationResult MyFriendlyMethod(int id, Customer customer)

    {

        return new OperationResult.Modified { ResponseResource =
customer };

    }

}

If you want to have a better control of the Http status code, as it is
shown in the code above, you can also return an instance of
OperationResult.

**Codecs**

As you can see, the handlers have not been wired to any content type at
this point. Serializing/Deserializing a resource instance into an
specific representation is responsibility of the codecs. Some codecs are
provided out of the box in OR, for example, Json, Xml, Html or
Form-Url-Encoded data.

A codec generally implement two interfaces, IMediaTypeReader (for
deserializing content) and IMediaTypeWriter(for serializing). The
following code shows the implementation of the Json codec,

[MediaType("application/json;q=0.5", "json")]

public class JsonDataContractCodec : IMediaTypeReader, IMediaTypeWriter

{

    public object Configuration { get; set; }

    public object ReadFrom(IHttpEntity request, System.Type
destinationType, string paramName)

    {

        DataContractJsonSerializer serializer = new
DataContractJsonSerializer(destinationType);

        return serializer.ReadObject(request.Stream);

    }

    public void WriteTo(object entity, IHttpEntity response, string[]
paramneters)

    {

        if (entity == null)

            return;

        DataContractJsonSerializer serializer = new
DataContractJsonSerializer(entity.GetType());

        serializer.WriteObject(response.Stream, entity);

    }

}

**Putting Handlers and Codecs together**

By means of a fluent configuration, we can later wire up an URI with the
corresponding handler and one or more codecs according to the
content-types that the service should support.

ConfigureServer(() =\> ResourceSpace.Has.ResourcesOfType\<Customer\>()

                          .AtUri("/customer/{id}")

                          .HandledBy\<CustomerHandler\>()

                         
.AndTranscodedBy\<OpenRasta.Codecs.JsonDataContractCodec\>()

                         
.AndBy\<OpenRasta.Codecs.XmlSerializerCodec\>());

This is one of the big differences with the ASP.NET MVC or the Web
Programming Model in WCF.

In the MVC, the supported content types are inherently tied to the
ActionResult returned by the controller action. Therefore, the action
implementation must be modified in order to support new content types. 
Or a more specialized implementation of an ActionResult is required,
which has to be smart enough to accommodate these changes.

Same thing happens with WCF, you have to either decorate the service
operation with the supported content-type (Only Json or Pox supported
today) or return a Message/Stream instance from the operation and write
the resource representation yourself in the operation implementation. As
I said, only Json/Pox are supported today, and adding new content types
is something complicated to do. You basically need to write a custom
IDispatchMessageFormatter extension, and a custom WebServiceHost to
replace the formatter provided out of the box. (This is the approach
taken in the WCF REST Starter kit for supporting form-url-encoded data).

**Pipeline Contributors**

Let’s go back to OpenRasta for a moment, “Handlers” and “Codecs” are not
the only supported extensibility points in this framework. All the
processing of incoming/outgoing messages is performed in a pipeline that
contains contributors or filters. A pipeline contributor perform simple
tasks such as initializing the security context, routing the messages or
processing exceptions to name a few. You can write your own
contributors, or customize the pipeline to support different
configurations or scenarios

**More information about OpenRasta** 

You can find more information about this framework in the [Sebastien’s
blog](http://serialseb.blogspot.com/). The current project status is
also available
[here](http://serialseb.blogspot.com/2008/10/openrasta-status-update.html).

