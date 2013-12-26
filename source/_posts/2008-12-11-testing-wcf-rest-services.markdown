---
layout: post
title: "Testing WCF REST Services"
date: 2008-12-11
comments: true
categories: REST
---

Many of the WCF services that we build today rely on the WCF context
(OperationContext and WebOperationContext) for performing different
things, specially REST services where the context is necessary for
settings and getting Http status codes or headers. The fact that the WCF
context does not expose interfaces or base classes complicates the unit
testing a lot. In this sense, I like the approach taken by the ASP.NET
MVC team, all the classes exposed by the HttpContext as properties are
base classes, so they can be easily mocked. \
In order to test a WCF service with what we have today, we have to
either test the service as a black box (integration tests, which
requires a lot of plumbing code to setup all the WCF infrastructure for
the test, channels, host, client, etc) or create some wrappers to
encapsulate the WCF context behavior, as I mentioned in this post ["Unit
tests for WCF
services"](http://weblogs.asp.net/cibrax/archive/2008/05/16/unit-tests-for-wcf.aspx)

Today I will discuss both approaches more in detail with some sample
code. The service implementation I will use here is a simple service
that given a category returns a feed with products associated to that
specific category.

[ServiceContract]

public interface IProductCatalog

{

    [WebGet(UriTemplate = "?category={category}")]

    [OperationContract]

    Atom10FeedFormatter GetProducts(string category);

}

 

public Atom10FeedFormatter GetProducts(string category)

{

    var items = new List\<SyndicationItem\>();

    foreach(var product in repository.GetProducts(category))

    {

        items.Add(new SyndicationItem()

        {

            Id = String.Format(CultureInfo.InvariantCulture,
"http://products/{0}", product.Id),

            Title = new TextSyndicationContent(product.Name),

            LastUpdatedTime = new DateTime(2008, 7, 1, 0, 0, 0,
DateTimeKind.Utc),

            Authors =

            {

                new SyndicationPerson()

                {

                    Name = "cibrax"

                }

            },

            Content = new TextSyndicationContent(string.Format("Category
Id {0} - Price {1}",

                product.Category, product.UnitPrice))

        });

    }

 

    var feed = new SyndicationFeed()

    {

        Id = "http://Products",

        Title = new TextSyndicationContent("Product catalog"),

        Items = items

    };

 

    MyWebContext.Current.OutgoingResponse.ContentType =
"application/atom+xml";

    return feed.GetAtom10Formatter();

}

As you can see, this is a regular WCF service with the following
characteristics,

​1. The products are queried from a product repository, a class that
implements IProductRepository so it can be mocked.

public interface IProductRepository

{

    IQueryable\<Product\> GetProducts(string category);

}

​2. It is not using the WCF context (WebOperationContext), it is using
MyWebContext instead. This is a custom class I created for wrapping the
WCF context behavior. As we will see later, this class will help us to
mock the WCF context in the unit tests.

public class MyWebContext : IDisposable

{

    private const string TlsName = "webContext";

 

    public MyWebContext(IWebOperationContext context)

    {

        var tls = Thread.GetNamedDataSlot(TlsName);

        Thread.SetData(tls, context);

    }

 

    public static IWebOperationContext Current

    {

        get

        {

            var tls = Thread.GetNamedDataSlot(TlsName);

            var context = Thread.GetData(tls);

 

            if (context != null)

            {

                return (IWebOperationContext)context;

            }

            else

            {

                return new
WebOperationContextWrapper(WebOperationContext.Current);

            }

        }

    }

 

    public void Dispose()

    {

        var tls = Thread.GetNamedDataSlot(TlsName);

        Thread.SetData(tls, null);

    }

}

The Current property of this class first tries to get an
IWebOperationContext from the thread local storage (That could be set by
the unit tests), and if none is available, it returns a wrapper to the
original WCF context.

**Integration Tests**

Integration tests focus more on testing the service as a whole, checking
not only the service behavior but also extensions in the WCF channel
stack and any other external dependency referenced by the service
itself. Integration tests are preferred over unit tests for certain
scenarios, if I want to test for instance how a service behaves with an
built-in extension for "conditional gets" in the WCF channel stack, that
can be easily done with an integration test.

Integration tests usually have the following structure:

​1. Some code to setup the WCF host, usually located at the beginning of
the test or as part of the test fixture initializer.

​2. The service invocation itself using a previously initialized web
http request or an specific WCF channel.

[TestMethod]

public void ShouldGetProductsFeed()

{

    ProductCatalog catalog = new ProductCatalog(

        new InMemoryProductRepository(

            new List\<Product\>{

            new Product { Id = "1", Category = "foo", Name = "Foo1",
UnitPrice = 1 },

            new Product { Id = "2", Category = "bar", Name = "bar2",
UnitPrice = 2 }

        }));

 

    WebServiceHost host = new WebServiceHost(catalog, new
Uri("http://localhost:7777/Products"));

 

    IEnumerable\<SyndicationItem\> items;

 

    try

    {

        host.Open();

 

        items = Consume(new
Uri("http://localhost:7777/Products?category=foo"));

    }

    finally

    {

        if (host.State == System.ServiceModel.CommunicationState.Opened)

            host.Close();

        else

            host.Abort();

    }

 

    Assert.AreEqual(1, items.Count());

    Assert.IsTrue(items.Any(i =\> i.Id == "http://products/1" &&
i.Title.Text == "Foo1"));

}

"Consume" in the code above is a helper method that internally uses
WebHttpRequest for sending a http get to the WCF service.

private IEnumerable\<SyndicationItem\> Consume(Uri feedUri)

{

    WebRequest request = CreateWebRequest(feedUri);

    XmlReaderSettings settings = new XmlReaderSettings { CloseInput =
true };

    using (XmlReader reader =
XmlReader.Create(request.GetResponse().GetResponseStream(), settings))

    {

        SyndicationFeed feed = SyndicationFeed.Load(reader);

        return feed.Items;

    }

}

 

private WebRequest CreateWebRequest(Uri feedUri)

{

    HttpWebRequest webRequest =
(HttpWebRequest)WebRequest.Create(feedUri);

    webRequest.ContentType = "application/rss+xml";

 

    webRequest.Timeout = 60 \* 1000;

 

    webRequest.Method = "GET";

 

    return webRequest;

}

**Unit Tests**

Unit tests focus on the service implementation only, leaving out most of
the external dependencies and any additional processing made by the WCF
channel stack. The IWebOperationContext interface I mentioned before
will help us to mock many of the properties provided by the WCF context.
This is a good thing for getting/setting expectations that we want to
verify in the test itself. This interface and the wrappers are available
as part of the project Netfx,
[http://code.google.com/p/netfx/](http://code.google.com/p/netfx/)

[TestClass]

public class UnitTests

{

    [TestMethod]

    public void ShouldGetProductsFeed()

    {

        ProductCatalog catalog = new ProductCatalog(

            new InMemoryProductRepository(

                new List\<Product\>{

                new Product { Id = "1", Category = "foo", Name = "Foo1",
UnitPrice = 1 },

                new Product { Id = "2", Category = "bar", Name = "bar2",
UnitPrice = 2 }

            }));

 

        Mock\<IWebOperationContext\> mockContext = new
Mock\<IWebOperationContext\> { DefaultValue = DefaultValue.Mock };

        IEnumerable\<SyndicationItem\> items;

 

        **using (new MyWebContext(mockContext.Object))**

**        {**

**            var formatter = catalog.GetProducts("foo");**

**            items = formatter.Feed.Items;**

**        }**

 

        mockContext.VerifySet(c =\> c.OutgoingResponse.ContentType,
"application/atom+xml");

 

        Assert.AreEqual(1, items.Count());

        Assert.IsTrue(items.Any(i =\> i.Id == "http://products/1" &&
i.Title.Text == "Foo1"));

    }

}

What's deserve some attention in the unit test above is the code for
creating the mocked WebContext and the code required for verifying the
expectations.

using (new MyWebContext(mockContext.Object))

{

    var formatter = catalog.GetProducts("foo");

    items = formatter.Feed.Items;

}

That will insert the mockContext object in the Thread Local Storage so
it can be accessed later in the service implementation by
MyWebContext.Current (The "using" clause here makes an explicit call to
IDispose as well, so the mockContext is removed from that storage after
the operation is called).  

The test also verifies that the ContentType header was set with the
correct value in the operation,

mockContext.VerifySet(c =\> c.OutgoingResponse.ContentType,
"application/atom+xml");

[Complete code available to download from
here](/images/legacy/WCF.Tests.zip)

