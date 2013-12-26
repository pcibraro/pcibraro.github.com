---
layout: post
title: "What I like about OpenRasta"
date: 2009-05-29
comments: true
categories: .NET
---

After having playing with the current bits of
[OpenRasta](http://serialseb.blogspot.com/2008/10/openrasta-status-update.html)
(OR) for a while, I  am now able to see great advantages in using this
framework over other existing frameworks for building RESTful services
such as WCF or the ASP.NET MVC.

Let’s explore some of the good features that make OpenRasta an excellent
alternative at the moment of developing RESTful services.

​1. Clean separation of resource representation from service
implementation

If we take a look at a service implementation in OR, the operations in a
handler receive or return a resource, but they never specify the final
representation of that resource. Resource representation is a pure
responsibility of the encoders configured for service implementation (a
handler in OR).

In that way, we have a service implementation completely reusable for
different resource representations.

public class CustomerHandler

{

    public OperationResult Get(int customerId)

    {

        return new OperationResult.OK

        {  

            ResponseResource = "Customer with id " + customerId

        };

    }

}

Once we have the service implementation, we only need to configure the
encoders for that service, which could be encoders for Xml, Json, Atom
or any other representation. You are free to configure all the encoders
you want for the same handler.

ConfigureServer(() =\> ResourceSpace.Has.ResourcesOfType\<Customer\>()

    .AtUri("/customer/{customerId}")

    .HandledBy\<CustomerHandler\>()

    .AndTranscodedBy\<OpenRasta.Codecs.JsonDataContractCodec\>(null));

This is a different story in WCF. First of all, the content type (Xml or
Json) has to be specified as part of the operation definition. If we
want to support different content types, we need to replicate the same
operation only to change the content type. Although this point has been
improved considerably in WCF 4.0, some service implementations are still
absolutely tied to the resource representation. For example, an
operation that returns syndications feeds. In WCF, we have to return a
concrete implementation of
System.ServiceModel.Syndication.SyndicationFeedFormatter. It is not a
simple as return a collection of entities or resources and specify
somewhere that we want to represent them as entries in a feed.

This works much better in ADO.NET data services, where you can just
return a resource or a collection of resources, and the framework itself
will chose the best representation according to what the client
initially sent in the “Accept” header. That representation could be Json
or Atom, nothing changes in the service implementation (which is also
automatically implemented). However, ADO.NET data services provide few
extensibility points today for injecting custom code, and that is
probably a good reason to move to OpenRasta.

​2. URI resolution

In OpenRasta, the URI resolution to a handler is also part of the
service configuration. This gives a great flexibility to have many URIs
that resolve to the same resource instance.

A typical example is a parent-child relationship, for instance, a
customer/orders relationship.

You might want to have different URIs to get access to the orders,

Customer/1/Orders/1 =\> Give me the order number 1 for the customer with
ID 1

Or you might want to access to the order directly

Orders/1 =\> Give me the order number 1

In OR, you have a single handler implementation for orders (order is the
resource in this case), and multiple URIs that resolve to an operation
in that particular handler.

ConfigureServer(() =\> ResourceSpace.Has.ResourcesOfType\<Order\>()

    .AtUri("/customer/{customerId}/Orders/{orderId}")

    .AndAt("/orders/{orderId}")

    .HandledBy\<OrderHandler\>()

    .AndTranscodedBy\<OpenRasta.Codecs.JsonDataContractCodec\>(null));

This is a problem in WCF because the URI template is tied to the
operation definition, if you want to have different URIs for the same
resource, you basically have to copy/paste the same operation just to
change the URIs.

[WebGet(UriTemplate="/Orders/{orderId}")]

public Order GetOrderById(int orderId)

{

  //implementation

}

[WebGet(UriTemplate = "/Customers/{customerId}/Orders/{orderId}")]

public Order GetCustomerOrder(int customerId, int orderId)

{

  //implementation

}

So, you will have as many operations as URI and content types you want
to support in the service.

