---
layout: post
title: "Injecting Custom Logic in ADO.NET Data Services"
date: 2009-06-08
comments: true
categories: .NET
---

ADO.NET Data services represent today one of the most powerful
alternatives to build RESTful services in the .NET platform. This
framework basically creates a RESTful API on top of any IQueryable data
source. Most of the steps required to publish a set of resources through
http and make them available for any client are automatically
implemented. 

Only Entity Framework data models are supported out of the box as
read/write data sources, and any other IQueryable data source is
considered by default read only. If you want to perform write operations
on any other data source different from Entity Framework, you have to
implement an additional interface IUpdatable as it is showed in this
[post](http://blogs.msdn.com/astoriateam/archive/2008/04/10/iupdatable-ado-net-data-services-framework.aspx).
(Update: The new CTP 1.5 seems to have a new class to implement custom
data providers, I have not had a chance to play with it yet)

Usually, you will need to inject custom logic in the services for
performing validations or implementing business rules.

ADO.NET services comes with two built-in extensibility points for adding
custom logic or code to a service, Service Operations and Interceptors.

Service operations represent a simple way to define custom read/write
views on top of the data source exposed by the service. It is intended
to be used in scenarios where you need to create complex queries, or
simple data aggregations that are not supported out of the box in
framework. If you want to expose those queries directly as another
resource in the service, you have to use service operations.

The classic example given to show this feature in action is a query that
returns all the customers in an specific city. That query would be
translated in a service operation as follow,

*public class MyService : DataService\<MyDataSource\> \
{ \
    // This method is called only once to initialize service-wide
policies. \
    public static void InitializeService(IDataServiceConfiguration
config) \
    { \
        config.SetEntitySetAccessRule("Customers",
EntitySetRights.AllRead); \
        config.SetServiceOperationAccessRule("CustomersInCity",
ServiceOperationRights.All); \
    }*

*    [WebGet] \
    public IQueryable\<MyDataSource.Customers\> CustomersInCity(string
city) \
    { \
        return from c in this.CurrentDataSource.Customers \
               where c.City == city \
               select c; \
    } *

*}*

As you can see in the code above, the service operation for this example
is a simple method decorated with a WebGet attribute that returns an
IQueryable implementation (With a pre-defined query in this example).
The WebGet attribute specifies that  is a read-only view. As soon as we
have that service running, you can navigate to that service operation
with an URI like this, **

*[http://localhost/MyService.svc/CustomersInCity?city=BuenosAires](http://localhost/MyService.svc/CustomersInCity?city=BuenosAires)*

Since you are returning an IQueryable implementation, you are also free
to define additional transformations on top of this base query, such as
projections, filtering, or ordering among others.

WebInvoke can be also used in case you want to implement a write
operation. Since a service operation works as a black box, only the post
verb is supported for WebInvoke, which means that you can not use other
verbs like DELETE or PUT. Regarding this limitation, this is the answer
you can find in the
[forums](http://social.msdn.microsoft.com/Forums/en-US/adodotnetdataservices/thread/e546042f-c5f7-489d-b3e7-03b84a4f07ab)

***“We currently support only POST and GET verbs on the service
operations (even in the recently released CTP). Is there a reason why
you want to support  DELETE verb on service operation? The main reason
for not supporting all the verbs is that from the client side, discovery
becomes a issue - there is no way to know what verb to send for a
service operation. Also service operations are black box to us - so we
try and categorized them into side-effecting and non-side effecting
ones. We recommend to use GET for non-side effection service opertions
and use POST for side-effecting ones.***

***Please let us know if you think there is a good reason why DELETE
verb should be supported.***

***Thanks \
Pratik”***

Another problem with service operations is that only scalar values are
supported as arguments. Therefore, you can not pass entities or complex
data types as arguments to a service operation.

These limitations make hard to use service operations as a way to inject
custom logic like validations or business rules into a data service.

Fortunately, there is another extensibility point that becomes handy at
the moment of implementing this kind of logic, the Interceptors. There
are two kind of built-in interceptors, Query Interceptors, and Change
Interceptors.

A query interceptor represents a mechanism to change or transform the
initial projection over an entity set. For example, if an user only
allowed to see a subset of all the available data in an entity set, a
query interceptor is the right place to filter the data for that user.

*[QueryInterceptor("Orders")] \
public Expression\<Func\<Orders, bool\>\> QueryOrders() { \
  return o =\> o.Number \> 100; \
}*

The syntax of a query interceptor is quite simple, you basically returns
an expression that describes a function (the Func), which takes an input
value (an order being evaluated) and returns a bool value (whether the
order passes the filter or not). Since an expression is used there
rather than a piece of code, that expression can be passed directly to
the query provider, which ultimately will generate the appropriate SQL
in the database server.

A change interceptor on the other hand, intercepts all the calls that
generates changes in the data source exposed by the service. Therefore,
this interceptor is the right mechanism for performing some validations
before the data is changed in the data source. This is the place where
we can enforce the business rules or validate the data for an specific
entity set in our application (Entity set in this scenario also
represents a collection of resources, where each resource is the entity
itself).

*[ChangeInterceptor("Orders")] \
public void OnChangeOrders(Order o, UpdateOperations operation) \
{ \
     if (operation == UpdateOperations.Add || operation ==
UpdateOperations.Change) \
     { \
        if (o.Number \> 100) \
        { \
          throw new DataServiceException(400, \
            "You are not allowed to create or change an order with
number greater than 100"); \
        } \
    } \
}*

A change interceptor method receives two arguments, the entity that will
be persisted, and an enumeration UpdateOperations that specifies the
operation currently performed on the entity. Possible values for this
enumeration are, Add, Change and Delete.

This interceptor is not just to perform validations, you are still free
to call additional services or change some data in the entity before it
is persisted. However, this method call is synchronous, so you should be
careful about executing long running code there.

