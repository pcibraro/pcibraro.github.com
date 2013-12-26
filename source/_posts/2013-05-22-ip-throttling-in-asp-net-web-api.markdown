---
layout: post
title: "IP Throttling in ASP.NET Web API"
date: 2013-05-22
comments: true
categories: .NET
---

Some Web APIs use the client IP address to enforce Service Level
Agreements such as limit the number of calls in a period of time. The
client IP address can be used as a replacement for an authentication key
sometimes when a previous registration of client applications is not
required.

This is relatively simple to implement in a message handler,

```csharp
public class IPThrottlingMessageHandler : DelegatingHandler
{
  IIPRepository repository;
  int maxRequestsHour;

  public IPThrottlingMessageHandler(IIPRepository repository, int maxRequestsHour = 150)
            : base()
  {
    this.repository = repository;
    this.maxRequestsHour = maxRequestsHour;
   }

   public IPThrottlingMessageHandler(HttpMessageHandler inner, IIPRepository repository, int maxRequestsHour = 150)
            : base(inner)
  {
     this.repository = repository;
     this.maxRequestsHour = maxRequestsHour;
   }

   protected override System.Threading.Tasks.Task<HttpResponseMessage> SendAsync(HttpRequestMessage request, System.Threading.CancellationToken cancellationToken)
  {
     var ip = GetClientIp(request);
            
      if (ip == null)
      {
         return ToResponse(request, HttpStatusCode.Forbidden, "The client ip couldn't be found");
       }

       if(this.repository.Increment(DateTime.Now.Hour, ip) > this.maxRequestsHour)
       {
          return ToResponse(request, HttpStatusCode.Forbidden, "Quota exceeded");
        }
            
       return base.SendAsync(request, cancellationToken);
  }

  private string GetClientIp(HttpRequestMessage request)
  {
     if (request.Properties.ContainsKey("MS_HttpContext"))
     {
        return ((HttpContextWrapper)request.Properties["MS_HttpContext"]).Request.UserHostAddress;
      }
     else if (request.Properties.ContainsKey(RemoteEndpointMessageProperty.Name))
     {
        RemoteEndpointMessageProperty prop;
        prop = (RemoteEndpointMessageProperty)request.Properties[RemoteEndpointMessageProperty.Name];
        return prop.Address;
      }
      else
      {
          return null;
       }
   }

   private static Task<HttpResponseMessage> ToResponse(HttpRequestMessage request, HttpStatusCode code, string message)
   {
       var tsc = new TaskCompletionSource<HttpResponseMessage>();

       var response = request.CreateResponse(code);
       response.ReasonPhrase = message;
       response.Content = new StringContent(message);

       tsc.SetResult(response);

        return tsc.Task;
    }
 }
```

This handler uses a repository to store the number of calls with a given
IP in one hour. If the number of requests per hour exceeds the quota, an
error response is returned to the client.

