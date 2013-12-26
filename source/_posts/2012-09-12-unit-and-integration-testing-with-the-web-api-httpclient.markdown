---
layout: post
title: "Doing unit and integration tests with the Web API HttpClient"
date: 2012-09-12
comments: true
categories: .NET
---

One of the nice things about the new HttpClient in System.Net.Http is
the support for mocking responses or handling requests in a http server
hosted in-memory. While the first option is useful for scenarios in
which we want to test our client code in isolation (unit tests for
example), the second one enables more complete integration testing
scenarios that could include some more components in the stack such as
model binders or message handlers for example.  

The HttpClient can receive a HttpMessageHandler as argument in one of
its constructors.

```csharp
public class HttpClient : HttpMessageInvoker
{
   public HttpClient();
   public HttpClient(HttpMessageHandler handler);
   public HttpClient(HttpMessageHandler handler, bool disposeHandler);
```

For the first scenario, you can create a new HttpMessageHandler that
fakes the response, which you can use in your unit test. The only
requirement is that you somehow inject an HttpClient with this custom
handler in the client code.

```csharp
public class FakeHttpMessageHandler : HttpMessageHandler
{
  HttpResponseMessage response;

  public FakeHttpMessageHandler(HttpResponseMessage response)
  {
       this.response = response;
   }

   protected override Task<HttpResponseMessage> SendAsync(HttpRequestMessage request, 
System.Threading.CancellationToken cancellationToken)
   {
      var tcs = new TaskCompletionSource<HttpResponseMessage>();
            
      tcs.SetResult(response);

      return tcs.Task;
    }
}
```

In an unit test, you can do something like this.

```csharp
var fakeResponse = new HttpResponse();
var fakeHandler = new FakeHttpMessageHandler(fakeResponse);
var httpClient = new HttpClient(fakeHandler);

var customerService = new CustomerService(httpClient);

// Do something

// Asserts
```

CustomerService in this case is the class under test, and the one that
receives an HttpClient initialized with our fake handler.

For the second scenario in integration tests, there is a In-Memory host
“System.Web.Http.HttpServer” that also derives from HttpMessageHandler
and you can use with a HttpClient instance in your test. This has been
discussed already in these two great posts from
[Pedro](http://pfelix.wordpress.com/2012/03/05/asp-net-web-api-in-memory-hosting/)
and
[Filip](http://www.strathweb.com/2012/06/asp-net-web-api-integration-testing-with-in-memory-hosting/). 

