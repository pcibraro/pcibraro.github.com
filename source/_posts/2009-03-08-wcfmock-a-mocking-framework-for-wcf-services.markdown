---
layout: post
title: "WCFMock, a mocking framework for WCF services"
date: 2009-03-08
comments: true
categories: Unit-Testing
---

WCFMock, a mocking framework for WCF services. Not a very original name,
but it was the first one that came out to my mind :). If you have been
following my blog for a while, you might notice that I discussed
different approaches in the past to unit test WCF services,
[here](http://weblogs.asp.net/cibrax/archive/2008/05/16/unit-tests-for-wcf.aspx)
and
[here](http://weblogs.asp.net/cibrax/archive/2008/12/11/testing-wcf-rest-services.aspx).
One of the major pains that you will find today for unit testing WCF
services is the static operation context (OperationContext and
WebOperationContext). If you service implementation relies on that
context for doing something, you will have a hard time trying to test
that functionality.

For instance, it is very common in WCF REST services to use the context
to set or get http status codes. With the current WCF bits, how can you
do to unit test those services ?. The answer is WCFMock, a set of useful
classes that will help you to remove all the explicit dependencies with
the operation context, and still provide a good way to mock them from
unit tests.

Let's see how WCFMock works in practice with a very simple example,

​1. You have a WCF REST service that returns a RSS feed with a catalog
of products

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

    WebOperationContext.Current.OutgoingResponse.ContentType =
"application/atom+xml";

    return feed.GetAtom10Formatter();

}

 

 

This service implementation only relies on the WebOperationContext to
set up the response content type, that is being done in the following
line,

 

WebOperationContext.Current.OutgoingResponse.ContentType =
"application/atom+xml";

 

​2. You have now to find a way to get rid of that dependency so we can
unit test that method. Here is where WCFMock comes to the rescue. The
first thing you have to do is to define a new alias on top of your
class,

using WebOperationContext =
System.ServiceModel.Web.MockedWebOperationContext;

Optionally, you can wrap that sentence with a conditional compilation
directive

\#if DEBUG

using WebOperationContext =
System.ServiceModel.Web.MockedWebOperationContext;

\#endif

This is useful for instance, if you want to use the mocked version in
development, and always the WCF version in production. That's all, you
do not need to touch your existing service implementation at all, once
you defined that alias, the service is ready to be tested.

​3. For testing the service, I will use Moq, a pretty good mock
framework created by my friend [Cazzu](http://weblogs.asp.net/cazzu).

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

        **using (new MockedWebOperationContext(mockContext.Object))**

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

Two pieces of code deserves some special attention, the code for
creating the mocked WebOperationContext and the code required for
verifying the expectations.

using (new MockedWebOperationContext(mockContext.Object))

{

    var formatter = catalog.GetProducts("foo");

    items = formatter.Feed.Items;

}

That will insert the mockContext object in the Thread Local Storage so
it can be accessed later in the service implementation.  

The test also verifies that the ContentType header was set with the
correct value in the operation,

mockContext.VerifySet(c =\> c.OutgoingResponse.ContentType,
"application/atom+xml");

As you can see, all the magic is done by the MockedWebOperationContext
(There is also a MockedOperationContext to replace the
OperationContext). The implementation of this class is quite simple,

public class MockedWebOperationContext : IDisposable

{

    [ThreadStatic]

    private static IWebOperationContext currentContext;

    public MockedWebOperationContext(IWebOperationContext context)

    {

        currentContext = context;

    }

    public static IWebOperationContext Current

    {

        get

        {

            if (currentContext == null)

            {

                return new
WebOperationContextWrapper(WebOperationContext.Current);

            }

            return currentContext;

        }

    }

    public void Dispose()

    {

        currentContext = null;

    }

}

The Current property of this class first tries to get an
IWebOperationContext from the thread local storage (That could be set by
the unit tests), and if none is available, it returns a wrapper to the
original WCF context. As you can see, the overhead introduced by the
class is minimal, it just a quick search in the TLS.

The project is now available to download from Codeplex at this location,
[http://wcfmock.codeplex.com/](http://wcfmock.codeplex.com/)

Enjoy.

