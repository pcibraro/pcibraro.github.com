---
layout: post
title: "Message Handlers per Route in ASP.NET Web API"
date: 2013-02-25
comments: true
categories: .NET
---

Message Handlers are one of the core components for message processing
in Web API. They use an asynchronous model for processing messages, so
they receive a request message and returns a Task with the corresponding
response.

```csharp
protected internal abstract Task<HttpResponseMessage> SendAsync(HttpRequestMessage request,
 CancellationToken cancellationToken);
```

In most cases, a message handler does something and delegates some other
work to the rest of the handlers configured in the pipeline. For
example, a handler for security checks the Auth Http header, and
delegates the call the handlers configured out of the box by Web API,
which eventually will call a controller method. The framework also
provides a base class to make delegation implicit, DelegatingHandler,
which receives the next handler to call as part of the constructor.

The following example shows a message handler implementation for basic
authentication,

```csharp
public class BasicAuthHandler : DelegatingHandler
{
   Func<string, string, IPrincipal> auth;

   public BasicAuthHandler(HttpMessageHandler innerHandler, Func<string, string, IPrincipal> auth)
       : base(innerHandler)
   {
       this.auth = auth;
    }

    protected override System.Threading.Tasks.Task<HttpResponseMessage> SendAsync(HttpRequestMessage request, 
```

```csharp
System.Threading.CancellationToken cancellationToken)
    {
         if (request.Headers.Authorization != null &&
             !string.Equals(request.Headers.Authorization.Scheme, "basic", 
```

```csharp
StringComparison.InvariantCultureIgnoreCase))
         {
              return base.SendAsync(request, cancellationToken);
          }

          if (request.Headers.Authorization == null ||
              string.IsNullOrWhiteSpace(request.Headers.Authorization.Scheme))
          {
              return ChallengeResponse(request);
           }

          var authToken = request.Headers.Authorization.Parameter;
          var decodedToken = Encoding.UTF8.GetString(Convert.FromBase64String(authToken));

          var username = decodedToken.Substring(0, decodedToken.IndexOf(":"));
          var password = decodedToken.Substring(decodedToken.IndexOf(":") + 1);

          var principal = this.auth(username, password);
            
          if (principal == null)
          {
              return ToResponse(request, HttpStatusCode.Unauthorized, "Invalid credentials");
           }

          Thread.CurrentPrincipal = principal;
          if (HttpContext.Current != null)
             HttpContext.Current.User = principal;
                        
          return base.SendAsync(request, cancellationToken);
   }

   private static Task<HttpResponseMessage> ChallengeResponse(HttpRequestMessage request)
   {
       var tsc = new TaskCompletionSource<HttpResponseMessage>();

       var response = request.CreateResponse(HttpStatusCode.Unauthorized);
        response.Headers.WwwAuthenticate.Add(new AuthenticationHeaderValue("basic", "realm=localhost"));

       tsc.SetResult(response);

       return tsc.Task;
     }

     private static Task<HttpResponseMessage> ToResponse(HttpRequestMessage request, 
```

```csharp
HttpStatusCode code, string message)
     {
        var tsc = new TaskCompletionSource<HttpResponseMessage>();

         var response = request.CreateResponse(code);
         response.ReasonPhrase = message;

         tsc.SetResult(response);

         return tsc.Task;
      }
 }
```

A good thing about Message Handler is that you can configure them
globally or per route. In this case, if you want to enable basic
authentication for some routes only, it’s a matter of configuring this
handler in the routes you want to have protected.

```csharp
config.Routes.MapHttpRoute(
   "BasicAuth",
   "MyController",
   new { controller = "MyController" },
   null,
   new BasicAuthHandler(new HttpControllerDispatcher(config), (u, p) => 
   {
      return new GenericPrincipal(new GenericIdentity(u, new string[] {}));
   }));
```

As you can see, the Inner Handler is a built-in handler provided by Web
API, HttpControllerDispatcher, which does all the magic for processing
the request and pass it over to the controller action. You can also
inject any other dependency as part of the constructor. One thing to
consider is that message handlers are singleton if you configure them
this way, so make sure to inject the dependencies in the right way for
avoiding memory leaks (If you have to use a repository for example, you
might want to inject a factory or pass a delegate for resolving the
dependencies from a DI container).

