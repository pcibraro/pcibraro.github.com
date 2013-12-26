---
layout: post
title: "Implementing a generic repository for WCF data services"
date: 2010-05-31
comments: true
categories: .NET
---

The repository implementation I am going to discuss here is not exactly
what someone would call repository in terms of DDD, but it is an
abstraction layer that becomes handy at the moment of unit testing the
code around this repository. In other words, you can easily create a
mock to replace the real repository implementation.

The WCF Data Services update for .NET 3.5 introduced a nice feature to
support two way data bindings, which is very helpful for developing WPF
or Silverlight based application but also for implementing the
repository I am going to talk about. As part of this feature, the WCF
Data Services Client library introduced a new collection
DataServiceCollection\<T\> that implements INotifyPropertyChanged to
notify the data context (DataServiceContext) about any change in the
association links. This means that it is not longer necessary to
manually set or remove the links in the data context when an item is
added or removed from a collection.

Before having this new collection, you basically used the following code
to add a new item to a collection.

Order order = new Order \
{ \
  Name = "Foo" \
};

OrderItem item = new OrderItem \
{ \
  Name = "bar", \
  UnitPrice = 10, \
  Qty = 1 \
};

var context = new OrderContext(); \
context.AddToOrders(order); \
context.AddToOrderItems(item);

context.SetLink(item, "Order", order);

context.SaveChanges();

Now, thanks to this new collection, everything is much simpler and
similar to what you have in other ORMs like Entity Framework or L2S.

Order order = new Order \
{ \
  Name = "Foo" \
};

OrderItem item = new OrderItem \
{ \
  Name = "bar", \
  UnitPrice = 10, \
  Qty = 1 \
};

order.Items.Add(item);

var context = new OrderContext(); \
context.AddToOrders(order);

context.SaveChanges();

In order to use this new feature, you first need to enable V2 in the
data service, and then use some specific arguments in the datasvcutil
tool (You can find more information about this new feature and how to
use it in this
[post](http://blogs.msdn.com/b/astoriateam/archive/2009/09/01/introduction-to-data-binding-in-ctp2.aspx)).

DataSvcUtil
/uri:"[http://localhost:3655/MyDataService.svc/](http://localhost:3655/MyDataService.svc/)"
/out:Reference.cs /dataservicecollection /version:2.0

Once you use those two arguments, the generated proxy classes will use
DataServiceCollection\<T\> rather than a simple ObjectCollection\<T\>,
which was the default collection in V1.

There are some aspects that you need to know to use this feature
correctly.

​1. All the entities retrieved directly from the data context with a
query track the changes and report those to the data context
automatically.

​2. A entity created with “new” does not track any change in the
properties or associations. In order to enable change tracking in this
entity, you need to do the following trick.

public Order CreateOrder() \
{ \
  var collection = new DataServiceCollection\<Order\>(this.context); \
  var order = new Order();

  collection.Add(order);

  return order; \
}

You basically need to create a collection, and add the entity to that
collection with the “Add” method to enable change tracking on that
entity.

​3. If you need to attach an existing entity (For example, if you
created the entity with the “new” operator rather than retrieving it
from the data context with a query) to a data context for tracking
changes, you can use the “Load” method in the DataServiceCollection.

var order = new Order \
{ \
  Id = 1 \
};

var collection = new DataServiceCollection\<Order\>(this.context); \
collection.Load(order);

In this case, the order with Id = 1 must exist on the data source
exposed by the Data service. Otherwise, you will get an error because
the entity did not exist.

These cool extensions methods discussed by Stuart Leeks in this
[post](http://blogs.msdn.com/b/stuartleeks/archive/2008/09/15/dataservicequery-t-expand.aspx)
to replace all the magic strings in the “Expand” operation with
Expression Trees represent another feature I am going to use to
implement this generic repository.

Thanks to these extension methods, you could replace the following query
with magic strings by a piece of code that only uses expressions.

Magic strings,

var customers = dataContext.Customers \
.Expand("Orders") \
        .Expand("Orders/Items")

Expressions,

var customers = dataContext.Customers \
.Expand(c =\> c.Orders.SubExpand(o =\> o.Items))

That query basically returns all the customers with their orders and
order items.

Ok, now that we have the automatic change tracking support and the
expression support for explicitly loading entity associations, we are
ready to create the repository.

The interface for this repository looks like this,

~~~~ {.code}
public interface IRepository
{
  T Create<T>() where T : new();
  void Update<T>(T entity);
  void Delete<T>(T entity);
  IQueryable<T> RetrieveAll<T>(params Expression<Func<T, object>>[] eagerProperties);
  IQueryable<T> Retrieve<T>(Expression<Func<T, bool>> predicate, 
~~~~

~~~~ {.code}
params Expression<Func<T, object>>[] eagerProperties);
  void Attach<T>(T entity);
  void SaveChanges();
}
~~~~

[](http://11011.net/software/vspaste)

The Retrieve and RetrieveAll methods are used to execute queries against
the data service context. While both methods receive an array of
expressions to load associations explicitly, only the Retrieve method
receives a predicate representing the “where” clause.

The following code represents the final implementation of this
repository.

~~~~ {.code}
public class DataServiceRepository: IRepository
{
  ResourceRepositoryContext context;

  public DataServiceRepository()
      : this (new DataServiceContext())
  {
  }

  public DataServiceRepository(DataServiceContext context)
  {
            this.context = context;
  }

  private static string ResolveEntitySet(Type type)
  {
    var entitySetAttribute = (EntitySetAttribute)type
~~~~

~~~~ {.code}
.GetCustomAttributes(typeof(EntitySetAttribute), true).FirstOrDefault();

   if (entitySetAttribute != null)
      return entitySetAttribute.EntitySet;

   return null;
  }

  public T Create<T>() where T : new()
  {
    var collection = new DataServiceCollection<T>(this.context);
    var entity = new T();

    collection.Add(entity);

    return entity;
  }

  public void Update<T>(T entity)
  {
    this.context.UpdateObject(entity);
  }

  public void Delete<T>(T entity)
  {
    this.context.DeleteObject(entity);
  }

  public void Attach<T>(T entity)
  {
    var collection = new DataServiceCollection<T>(this.context);
    collection.Load(entity);
  }

  public IQueryable<T> Retrieve<T>(Expression<Func<T, bool>> predicate, 
~~~~

~~~~ {.code}
  params Expression<Func<T, object>>[] eagerProperties)
  {
    var entitySet = ResolveEntitySet(typeof(T));
    var query = context.CreateQuery<T>(entitySet);

    foreach (var e in eagerProperties)
    {
      query = query.Expand(e);
    }

    return query.Where(predicate);
  }

  public IQueryable<T> RetrieveAll<T>(params Expression<Func<T, object>>[] eagerProperties)
  {
    var entitySet = ResolveEntitySet(typeof(T));
    var query = context.CreateQuery<T>(entitySet);

    foreach (var e in eagerProperties)
    {
      query = query.Expand(e);
    }

    return query;
  }

  public void SaveChanges()
  {
    this.context.SaveChanges(SaveChangesOptions.Batch);
  }

 }
~~~~

[](http://11011.net/software/vspaste)

For instance, you can use the following code to retrieve customers with
First name equal to “John”, and all their orders in a single call.

repository.Retrieve\<Customer\>( \
   c =\> c.FirstName == “John”, //Where \
   c =\> c.Orders.SubExpand(o =\> o.Items));

In case, you want to have some pre-defined queries that you are going to
use across several places, you can put them in an specific class.

public static class CustomerQueries \
{ \
  public static Expression\<Func\<Customer, bool\>\>
LastNameEqualsTo(string lastName) \
  { \
    return c =\> c.LastName == lastName; \
  } \
}

And then, use it with the repository.

repository.Retrieve\<Customer\>( \
   CustomerQueries.LastNameEqualsTo("foo"), \
   c =\> c.Orders.SubExpand(o =\> o.Items));

