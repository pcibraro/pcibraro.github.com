---
layout: post
title: "Unit tests for WCF (And Moq)"
date: 2008-05-16
comments: true
categories: Moq
---

As you may know, testing WCF services is not as simple as referencing a
service implementation and start writing unit tests against it. If the
service we want to test has a high dependency with the operation
context, which is an static class, testing that service can be a very
complicated task.

Many times that dependency can be removed thanks to the help of some WCF
extensibility points like behaviors, authorization managers or
authorization policies to name a few. For instance, if an operation
implementation in our service is looking for specific security claims to
check user permissions, that code could be moved to an authorization
manager to make the resulting code easier to test.

However, there are some cases where the dependency with the context or
other objects can not be completely removed (Or it is not solution), and
therefore the Dependency Injection pattern comes to help us. I have
already discussed how to use [DI in
WCF](http://weblogs.asp.net/cibrax/archive/2007/12/13/wcf-dependency-injection-behavior.aspx)
before, and [other
people](http://javicrespotech.blogspot.com/2008/05/dependency-injection-in-wcf-services.html)
have also done the same.

Ok, let's say that we are in that scenario and we want to use DI to
remove all the dependencies of our service with the WCF operation
context. How can we do that ?. The WCF context is neither a base class
nor an interface, we can not mock it at all at first glance.
Fortunately, there is a easy workaround to solve this problem, it
consists of creating a public interface with all the properties and
methods that we want to use (and mock) from the WCF context.

public interface IOperationContext

{

     event EventHandler OperationCompleted;

 

     // Methods

     T GetCallbackChannel\<T\>();

     void SetTransactionComplete();

 

     // Properties

     IContextChannel Channel { get; }

     IExtensionCollection\<OperationContext\> Extensions { get; }

     bool HasSupportingTokens { get; }

     ServiceHostBase Host { get; }

     MessageHeaders IncomingMessageHeaders { get; }

     MessageProperties IncomingMessageProperties { get; }

     MessageVersion IncomingMessageVersion { get; }

     InstanceContext InstanceContext { get; }

     bool IsUserContext { get; }

     MessageHeaders OutgoingMessageHeaders { get; }

     MessageProperties OutgoingMessageProperties { get; }

     RequestContext RequestContext { get; set; }

     IServiceSecurityContext ServiceSecurityContext { get; }

     string SessionId { get; }

     ICollection\<SupportingTokenSpecification\> SupportingTokens { get;
}

}

Same thing can be done for the ServiceSecurityContext and
WebOperationContext.

public interface IServiceSecurityContext

{

    AuthorizationContext AuthorizationContext { get; }

    ReadOnlyCollection\<IAuthorizationPolicy\> AuthorizationPolicies {
get; }

    bool IsAnonymous { get; }

    IIdentity PrimaryIdentity { get; }

    WindowsIdentity WindowsIdentity { get; }

}

 

public interface IWebOperationContext

{

  IIncomingWebRequestContext IncomingRequest { get; }

  IOutgoingWebResponseContext OutgoingResponse { get; }

}

 

Secondly, we need a default implementation to wrap the existing WCF
context functionality, this wrapper will only delegates method calls to
the original context.

public class OperationContextWrapper : IOperationContext

  {

    OperationContext context;

 

    public OperationContextWrapper(OperationContext context)

    {

      this.context = context;

    }

 

    \#region IOperationContext Members

 

    public event EventHandler OperationCompleted;

 

    public T GetCallbackChannel\<T\>()

    {

      return context.GetCallbackChannel\<T\>();

    }

 

    public void SetTransactionComplete()

    {

      context.SetTransactionComplete();

    }

 

    public IContextChannel Channel

    {

      get { return context.Channel; }

    }

 

    public IExtensionCollection\<OperationContext\> Extensions

    {

      get { return context.Extensions; }

    }

 

    public bool HasSupportingTokens

    {

      get { return context.HasSupportingTokens; }

    }

 

    public ServiceHostBase Host

    {

      get { return context.Host; }

    }

 

    public System.ServiceModel.Channels.MessageHeaders
IncomingMessageHeaders

    {

      get { return context.IncomingMessageHeaders; }

    }

 

    public System.ServiceModel.Channels.MessageProperties
IncomingMessageProperties

    {

      get { return context.IncomingMessageProperties; }

    }

 

    public System.ServiceModel.Channels.MessageVersion
IncomingMessageVersion

    {

      get { return context.IncomingMessageVersion; }

    }

 

    public InstanceContext InstanceContext

    {

      get { return context.InstanceContext; }

    }

 

    public bool IsUserContext

    {

      get { return context.IsUserContext; }

    }

 

    public System.ServiceModel.Channels.MessageHeaders
OutgoingMessageHeaders

    {

      get { return context.OutgoingMessageHeaders; }

    }

 

    public System.ServiceModel.Channels.MessageProperties
OutgoingMessageProperties

    {

      get { return context.OutgoingMessageProperties; }

    }

 

    public System.ServiceModel.Channels.RequestContext RequestContext

    {

      get

      {

        return context.RequestContext;

      }

      set

      {

        context.RequestContext = value;

      }

    }

 

    public IServiceSecurityContext ServiceSecurityContext

    {

      get { return new
ServiceSecurityContextWrapper(context.ServiceSecurityContext); }

    }

 

    public string SessionId

    {

      get { return context.SessionId; }

    }

 

    public
ICollection\<System.ServiceModel.Security.SupportingTokenSpecification\>
SupportingTokens

    {

      get { return context.SupportingTokens; }

    }

 

    \#endregion

  }

 

public class ServiceSecurityContextWrapper : IServiceSecurityContext

  {

    ServiceSecurityContext context;

 

    public ServiceSecurityContextWrapper(ServiceSecurityContext context)

    {

      this.context = context;

    }

 

    \#region IServiceSecurityContext Members

 

    public System.IdentityModel.Policy.AuthorizationContext
AuthorizationContext

    {

      get

      {

        return this.context.AuthorizationContext;

      }

    }

 

    public ReadOnlyCollection\<IAuthorizationPolicy\>
AuthorizationPolicies

    {

      get

      {

        return this.context.AuthorizationPolicies;

      }

    }

 

    public bool IsAnonymous

    {

      get

      {

        return this.context.IsAnonymous;

      }

    }

 

    public System.Security.Principal.IIdentity PrimaryIdentity

    {

      get

      {

        return this.context.PrimaryIdentity;

      }

    }

 

    public System.Security.Principal.WindowsIdentity WindowsIdentity

    {

      get

      {

        return this.context.WindowsIdentity;

      }

    }

 

    \#endregion

  }

 

public class WebOperationContextWrapper : IWebOperationContext

  {

    private WebOperationContext context;

 

    public WebOperationContextWrapper(WebOperationContext context)

    {

      this.context = context;

    }

 

    \#region IWebOperationContext Members

 

    public IIncomingWebRequestContext IncomingRequest

    {

      get { return new
IncomingWebRequestContextWrapper(context.IncomingRequest); }

    }

 

    public IOutgoingWebResponseContext OutgoingResponse

    {

      get { return new
OutgoingWebResponseContextWrapper(context.OutgoingResponse); }

    }

 

    \#endregion

  }

Thirdly, the context must be injected as a dependency into the service.
This can be done through a constructor or a property setter, (This is
what our Unit test is going to use). If you do not want to deal with a
DI framework at all because it can complicates the service
implementation somehow, the service can have a default constructor to
set up the default context implementation (the wrapper). This is the
constructor that WCF will call by default when the service is created.
In the example bellow, I will show how to do that for a simple REST
service that uses the WCF WebOperationContext.

[ServiceContract]

public interface ICustomerService

{

    [OperationContract]

    [WebGet(UriTemplate = "/customers/{id}")]

    Customer FindCustomer(string id);

}

 

public class CustomerService : ICustomerService

{

  IWebOperationContext context;

 

  public CustomerService()

  {

    context = new
WebOperationContextWrapper(WebOperationContext.Current);

  }

 

  public CustomerService(IWebOperationContext context)

  {

    this.context = context;

  }

Now that we have all the infrastructure in place, we only need a good
mocking framework that helps us to create the mock objects for our
tests. In this part, I should recommend
[Moq](http://code.google.com/p/moq/) as the framework to use :). My
friend [Daniel Cazzulino](http://weblogs.asp.net/cazzu) has made an
excellent work developing this framework.

The pattern for using Moq or other similar framework in a unit test is
the following:

​1. You create a mock object from an existing interface or base class.
(The framework provides a way to do this).

​2. You set the expectations for that mock object. This is what you want
to be called, returned, set, etc.

​3. You pass the mock object to the object under testing.

​4. You call some methods on the object under testing, and afterwards,
you verify that all the expectations were met.

An interesting thing that we can do with Moq is to mock the complete
object graph of the WebOperationContext so we can reuse across all our
tests.

public class MockWebContext : Mock\<IWebOperationContext\>

{

    Mock\<IIncomingWebRequestContext\> requestContextMock = new
Mock\<IIncomingWebRequestContext\>();

    Mock\<IOutgoingWebResponseContext\> responseContextMock = new
Mock\<IOutgoingWebResponseContext\>();

 

    public MockWebContext()

      : base()

    {

      this.ExpectGet(webContext =\>
webContext.OutgoingResponse).Returns(responseContextMock.Object);

      this.ExpectGet(webContext =\>
webContext.IncomingRequest).Returns(requestContextMock.Object);

 

      WebHeaderCollection requestHeaders = new WebHeaderCollection();

      WebHeaderCollection responseHeaders = new WebHeaderCollection();

      UriTemplateMatch uriTemplateMatch = new UriTemplateMatch();

 

      requestContextMock.ExpectGet(requestContext =\>
requestContext.Headers).Returns(requestHeaders);

      requestContextMock.ExpectGet(requestContext =\>
requestContext.UriTemplateMatch).Returns(uriTemplateMatch);

      responseContextMock.ExpectGet(responseContext =\>
responseContext.Headers).Returns(responseHeaders);

    }

 

    public Mock\<IIncomingWebRequestContext\> IncomingRequest

    {

      get { return requestContextMock; }

    }

 

    public Mock\<IOutgoingWebResponseContext\> OutgoingResponse

    {

      get { return responseContextMock; }

    }

 

    public override void Verify()

    {

      base.Verify();

      requestContextMock.Verify();

      responseContextMock.Verify();

    }

 

    public override void VerifyAll()

    {

      base.VerifyAll();

      requestContextMock.VerifyAll();

      responseContextMock.VerifyAll();

    }

}

As you can see in that code, I basically created a mock for the
IWebOperationContext that also returns mock objects for the inner
properties like the IncomingRequest or OutgoingResponse. All the
expectations like default collections or contexts were set in the
constructor. We will use this mock later in the unit tests for the REST
service.

Our REST service implementation is quite simple, it only looks for a
customer with an specific identifier.

public Customer FindCustomer(string id)

{

   if (context.IncomingRequest.ContentType != "application/xml")

   {

      context.OutgoingResponse.StatusCode =
System.Net.HttpStatusCode.BadRequest;

      context.OutgoingResponse.SuppressEntityBody = true;

      return null;

   }

 

   if (id == "1")

      return new Customer { FirstName = "John", LastName = "Doe",
Address = "Address" };

 

   context.OutgoingResponse.StatusCode =
System.Net.HttpStatusCode.NotFound;

   context.OutgoingResponse.SuppressEntityBody = true;

 

   return null;

}

 

The service is expecting a "ContentType" http header equal to
"application/xml", and returning a "Not Found" status code if the
customer does not exists.

We can now write some unit tests to verify that our service is working
as we initially expected. The first one will verify that a valid
"Customer" instance is returned by the service is the "ContentType"
header is set correctly, and the right customer id is passed as argument
to the FindCustomer operation.

[Test]

public void ShouldReturnCustomer()

{

  MockWebContext mock = new MockWebContext();

  mock.IncomingRequest.ExpectGet(incomingRequest =\>
incomingRequest.ContentType).Returns("application/xml");

 

  CustomerService service = new CustomerService(mock.Object);

  Customer customer = service.FindCustomer("1");

 

  Assert.IsNotNull(customer);

}

The most important parts of the test above are:

​1. If the service ask for the content type, it should get
"application/xml".

mock.IncomingRequest.ExpectGet(incomingRequest =\>
incomingRequest.ContentType).Returns("application/xml");

​2. The mock object is passed as argument to the service implementation.

CustomerService service = new CustomerService(mock.Object);

 

We can create a second test to verify that the status code is set to
"NotFound" when an invalid customer id is passed as argument to the
FindCustomer operation.

[Test]

public void ShouldSetStatusCodeIfCustomerNotFound()

{

   MockWebContext mock = new MockWebContext();

   mock.IncomingRequest.ExpectGet(incomingRequest =\>
incomingRequest.ContentType).Returns("application/xml");

   mock.OutgoingResponse.ExpectSet(outgoingResponse =\>
outgoingResponse.StatusCode).Callback(sc =\>
Assert.AreEqual(HttpStatusCode.NotFound, sc));

 

   CustomerService service = new CustomerService(mock.Object);

   Customer customer = service.FindCustomer("5");

}

\
We are setting an expectation that the StatusCode should be "NotFound"
when the customer id is not equal to "1". (This is done in Moq by means
of a Callback).

mock.OutgoingResponse.ExpectSet(outgoingResponse =\>
outgoingResponse.StatusCode).Callback(sc =\>
Assert.AreEqual(HttpStatusCode.NotFound, sc));

Moq really simplifies everything, including the testing of services that
depends on the WCF operation context :).

Another approach to test WCF services, as
[Ploeh](http://blogs.msdn.com/ploeh) mentioned in this
[post](http://blogs.msdn.com/ploeh/archive/2006/12/04/IntegrationTestingWcfServices.aspx),
is what he call integration testing. If you do integration testing, you
are basically executing the service as a black box (the unit test in
this case plays the role of the client application) using all the
channel infrastructure provided by WCF. As you can notice, this approach
will require a lot of plumbing code and configuration to host the
service and set up the channels elements (Bindings and Behaviors).

