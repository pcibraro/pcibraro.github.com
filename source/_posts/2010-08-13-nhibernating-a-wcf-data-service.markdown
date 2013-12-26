---
layout: post
title: "NHibernating a WCF Data Service"
date: 2010-08-13
comments: true
categories: .NET
---

WCF Data Services ships with two built-in query providers, a query
provider for Entity Framework that uses the CSL model to infer all the
service metadata for the exposed entities and their associations, and
another provider, a Reflection Provider that uses .NET reflection over
the exposed object model to infer the same metadata.

The Entity Framework provider is usually the one that most people use
for its simplicity. The simplicity of this provider resides on the fact
that you actually don’t need to do much for getting a data service up
and running. You only need to define a entity framework model and expose
it in the data service using the DataService\<T\> class.

The Reflection Provider on the other hand requires some more work, as
you also need to implement the IUpdatable provider if you want to make
your model read-write (otherwise, it’s read-only by default).

While the Entity framework provider is simple to use, the resources you
want to expose in the data service gets tied to all the limitations you
find in an Entity Framework model (for instance, you might entities or
properties you don’t really want to persist in a database). This
provider also implements IUpdatable and that implementation can not be
customized, extended or replaced for providing some additional business
logic functionality in the data service. And although you can use
interceptors, I find that technique very limited as they represents
aspects that are applied to a single entity. There is no way to inject
cross-cutting aspects that affect all the entities (Entity based
Authorization for instance). You can use the Data Service Pipeline for
injecting that logic, but you don’t have entities at that point, only
the messages on the wire (atom feeds or json messages). A technique I
used in the past was to extend the EF entities with partial classes to
add some additional business logic to the entities, and attach the data
service to the Saving event in the Entity framework to run some logic
before the entities were created, updated or deleted. However, I still
don’t like this technique much because you end up with a model totally
limited to what you can define in EF.

The advantage of using the reflection provider is that you can expose a
much rich object model in your data service, even if you need to write
some more code. In addition, as you are also writing an IUpdatable
implementation, you can inject all the business logic or cross-cutting
concerns that are common for all the entities in that class. However,
the problem with the reflection provider is to make the service
implementation efficient enough to resolve the queries in the data
source and not the memory. It does not make sense at all to use a rich
object model on top of entity framework for instance, if you still need
to load all the object in memory to use linq to objects to perform the
queries (unless the number of entities you manage is really small). So,
the only possible solution to implement a service that manages a large
number of entities and expose a rich object model at the same time is to
use an ORM other than EF, like Linq to SQL or NHibernate.

NHibernate in that sense is a much mature framework, you are not tied to
a single db implementation (sql server), and the number of features that
this framework can offer is obviously higher, making NHibernate a good
technology for implementing data services. In addition, NHibernate 3.0
already ships with a Linq provider out of the box that works really well
(Entity Framework Code Only looks promising too but it is a CTP at this
point).

In order to use NHibernate in a data service, you need to provide an
IUpdatable implementation for making it read/write (support for POST PUT
and DELETE). Otherwise it will behave as read only by default (only GETs
supported).

This is how the IUpdatable implementation looks like,

~~~~ {#codeSnippet style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
public abstract class NHibernateDataContext : IUpdatable{    protected ISession Session;    private List<object> entityToUpdate = new List<object>();    private List<object> entityToRemove = new List<object>();    public NHibernateDataContext(ISession session)    {        this.Session = session;    }    /// <summary>    /// Creates the resource of the given type and belonging to the given container    /// </summary>    /// <param name="containerName">container name to which the resource needs to be added</param>    /// <param name="fullTypeName">full type name i.e. Namespace qualified type name of the resource</param>    /// <returns>object representing a resource of given type and belonging to the given container</returns>    object IUpdatable.CreateResource(string containerName, string fullTypeName)    {        Type t = Type.GetType(fullTypeName, true);        object resource = Activator.CreateInstance(t);                entityToUpdate.Add(resource);                return resource;    }    /// <summary>    /// Gets the resource of the given type that the query points to    /// </summary>    /// <param name="query">query pointing to a particular resource</param>    /// <param name="fullTypeName">full type name i.e. Namespace qualified type name of the resource</param>    /// <returns>object representing a resource of given type and as referenced by the query</returns>    object IUpdatable.GetResource(IQueryable query, string fullTypeName)    {        object resource = null;        foreach (object item in query)        {            if (resource != null)            {                throw new DataServiceException("The query must return a single resource");            }            resource = item;        }        if (resource == null)            throw new DataServiceException(404, "Resource not found");        // fullTypeName can be null for deletes        if (fullTypeName != null && resource.GetType().FullName != fullTypeName)            throw new Exception("Unexpected type for resource");        return resource;    }    /// <summary>    /// Resets the value of the given resource to its default value    /// </summary>    /// <param name="resource">resource whose value needs to be reset</param>    /// <returns>same resource with its value reset</returns>    object IUpdatable.ResetResource(object resource)    {        return resource;    }    /// <summary>    /// Sets the value of the given property on the target object    /// </summary>    /// <param name="targetResource">target object which defines the property</param>    /// <param name="propertyName">name of the property whose value needs to be updated</param>    /// <param name="propertyValue">value of the property</param>    void IUpdatable.SetValue(object targetResource, string propertyName, object propertyValue)    {        var propertyInfo = targetResource.GetType().GetProperty(propertyName);        propertyInfo.SetValue(targetResource, propertyValue, null);        if (!entityToUpdate.Contains(targetResource))            entityToUpdate.Add(targetResource);    }    /// <summary>    /// Gets the value of the given property on the target object    /// </summary>    /// <param name="targetResource">target object which defines the property</param>    /// <param name="propertyName">name of the property whose value needs to be updated</param>    /// <returns>the value of the property for the given target resource</returns>    object IUpdatable.GetValue(object targetResource, string propertyName)    {        var propertyInfo = targetResource.GetType().GetProperty(propertyName);        return propertyInfo.GetValue(targetResource, null);    }    /// <summary>    /// Sets the value of the given reference property on the target object    /// </summary>    /// <param name="targetResource">target object which defines the property</param>    /// <param name="propertyName">name of the property whose value needs to be updated</param>    /// <param name="propertyValue">value of the property</param>    void IUpdatable.SetReference(object targetResource, string propertyName, object propertyValue)    {        ((IUpdatable)this).SetValue(targetResource, propertyName, propertyValue);    }    /// <summary>    /// Adds the given value to the collection    /// </summary>    /// <param name="targetResource">target object which defines the property</param>    /// <param name="propertyName">name of the property whose value needs to be updated</param>    /// <param name="resourceToBeAdded">value of the property which needs to be added</param>    void IUpdatable.AddReferenceToCollection(object targetResource, string propertyName, object resourceToBeAdded)    {        PropertyInfo pi = targetResource.GetType().GetProperty(propertyName);        if (pi == null)            throw new Exception("Can't find property");        IList collection = (IList)pi.GetValue(targetResource, null);        collection.Add(resourceToBeAdded);        if (!entityToUpdate.Contains(targetResource))            entityToUpdate.Add(targetResource);    }    /// <summary>    /// Removes the given value from the collection    /// </summary>    /// <param name="targetResource">target object which defines the property</param>    /// <param name="propertyName">name of the property whose value needs to be updated</param>    /// <param name="resourceToBeRemoved">value of the property which needs to be removed</param>    void IUpdatable.RemoveReferenceFromCollection(object targetResource, string propertyName, object resourceToBeRemoved)    {        PropertyInfo pi = targetResource.GetType().GetProperty(propertyName);        if (pi == null)            throw new Exception("Can't find property");        IList collection = (IList)pi.GetValue(targetResource, null);        collection.Remove(resourceToBeRemoved);        if (!entityToUpdate.Contains(targetResource))            entityToUpdate.Add(targetResource);    }    /// <summary>    /// Delete the given resource    /// </summary>    /// <param name="targetResource">resource that needs to be deleted</param>    void IUpdatable.DeleteResource(object targetResource)    {        entityToRemove.Add(targetResource);    }    /// <summary>    /// Saves all the pending changes made till now    /// </summary>    void IUpdatable.SaveChanges()    {        using (var transaction = Session.BeginTransaction())        {            Session.FlushMode = FlushMode.Commit;            foreach (var entity in entityToUpdate)            {                Session.SaveOrUpdate(entity);            }            foreach (var entity in entityToRemove)            {                Session.Delete(entity);            }            transaction.Commit();        }            }    /// <summary>    /// Returns the actual instance of the resource represented by the given resource object    /// </summary>    /// <param name="resource">object representing the resource whose instance needs to be fetched</param>    /// <returns>The actual instance of the resource represented by the given resource object</returns>    object IUpdatable.ResolveResource(object resource)    {        return resource;    }    /// <summary>    /// Revert all the pending changes.    /// </summary>    void IUpdatable.ClearChanges()    {    }}
~~~~

\
This implementation receives an instance of a NHibernate session, and
implements all the required methods for creating, updating and deleting
existing entities. As you can see, this is an abstract class that you
can reuse for any NHibernate Data service. The concrete implementation
is the one that you will need to expose in the data service, and it is
tied to your entities. For instance,

 

~~~~ {#codeSnippet style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
public class MyNHibernateDataContext : NHibernateDataContext{    public MyNHibernateDataContext(ISession session)        : base(session)    {    }    public IQueryable<Customer> Customers    {        get        {            return new NhQueryable<Customer>(Session);                    }    }    public IQueryable<Person> People    {        get        {            return new NhQueryable<Person>(Session);        }    }}
~~~~

\

Finally, the data service exposes the concrete implementation of the
data context, and provides some code to initialize the NHibernate
session.

~~~~ {#codeSnippet style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
[ServiceBehavior(IncludeExceptionDetailInFaults=true)]public class MyDataService : DataService<MyNHibernateDataContext>, IDisposable{    public static void InitializeService(DataServiceConfiguration config)    {        config.SetEntitySetAccessRule("*", EntitySetRights.All);        config.DataServiceBehavior.AcceptCountRequests = true;        config.DataServiceBehavior.AcceptProjectionRequests = true;        config.DataServiceBehavior.MaxProtocolVersion = DataServiceProtocolVersion.V2;        config.UseVerboseErrors = true;    }    ISession session;    protected override MyNHibernateDataContext CreateDataSource()    {        var factory = CreateSessionFactory();        this.session = factory.OpenSession();        this.session.FlushMode = FlushMode.Auto;        return new MyNHibernateDataContext(this.session);    }    private static ISessionFactory CreateSessionFactory()    {        return Fluently.Configure()            .Database(MsSqlConfiguration.MsSql2008            .ConnectionString("Data source=.\\SQLExpress;Initial Catalog=Samples;Trusted_Connection=yes")            .Cache(c => c                .UseQueryCache()                .ProviderClass<HashtableCacheProvider>())            .ShowSql())            .Mappings(m => m.FluentMappings.AddFromAssemblyOf<Program>())            .BuildSessionFactory();    }
~~~~

\

This example uses Fluent NHibernate for mapping the entities to the
database structure. All the code for initializing the NHibernate session
is in the CreateDataSource method that the DataService calls for serving
any Http Request.  

 

The mapping for the entity “Customer” looks using Fluent NHibernate
looks as follow,

 

~~~~ {#codeSnippet style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
[DataServiceKey("Id")]public class Customer {    public virtual Guid Id { get; private set; }    public virtual string FullName { get; set; }    public virtual string Country { get; set; }    public virtual IList<Person> People { get; set; }    public virtual void AddPerson(Person p)    {        p.Customer = this;        this.People.Add(p);    }}
~~~~

\

~~~~ {#codeSnippet style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
public class CustomerMap : ClassMap<Customer>{    public CustomerMap()    {        Id(x => x.Id).GeneratedBy.GuidComb()           .UnsavedValue("00000000-0000-0000-0000-000000000000");          Map(x => x.FullName);        Map(x => x.Country);        HasMany(x => x.People)            .KeyColumn("CustomerId")            .Inverse()            .Cascade.All();    }}
~~~~

\

The DataServiceKey in the entity is something required by the reflection
provider for resolving queries by key (The dataservice will throw
exceptions if you don’t provide that attribute for your entities).

 

UPDATE!!: One colleague asked me an interesting question after I
submitted this post. How do you deal with lazy properties or collections
on the client side ?. This is not a problem at all with WCF Data
services, as the default behavior is to retrieve the entities that you
only asked for. That means that the associations (the lazy properties
and collections) are not retrieved unless you ask for them explicitly
with an expand (You are always in control of what you actually want to
get). The code on the client side that uses an expand is showed here,

 

~~~~ {#codeSnippet style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
var context = new MyNHibernateDataContext(new Uri("http://localhost:8080"));var customer = context.Customers  .Expand("People")  .First();
~~~~

\

Of course, this technique might not be optimal if you perform several
expands as you will be reproducing the common N+1 problem. I think the
only solution in that case is to implement a custom query provider that
knows how to submit a single query for returning all the associations at
once rather than performing different queries for each association.

