---
layout: post
title: "Giving temporary access to your ASP.NET Web API with Hawk"
date: 2013-03-08
comments: true
categories: .NET
---

One of the features supported by Hawk, an HTTP authentication protocol
based on HMAC, is to provide read-only access to a Web API for a short
period time.  That’s performed through a token called “bewit” that a Web
API can provide to a client. That token is only valid for Http GET calls
and it can be used for a limited period of time.

I already implemented this feature in my [Hawk port for
.NET](https://github.com/pcibraro/hawknet). A bewit token can be
generated as it is shown below,

```csharp
var credential = new HawkCredential
{
     Id = "dh37fgj492je",
     Key = "werxhqb98rpaxn39848xrunpaw3489ruxnpa98w4rxn",
     Algorithm = "hmacsha256",
     User = "steve"
 };

var bewit = Hawk.GetBewit("localhost", 
  new Uri("http://localhost:8091/Api/HelloWorld"), 
  credential, 
  60000);
```

The GetBewit method expects the following arguments,

-   The host name
-   The complete request URI
-   The Hawk credentials with information about the key and algorithm to
    use
-   A time-to-live setting in seconds for the token

That token is an string representation that you can add as a additional
query string in the Web API call.

```csharp
new HttpRequestMessage(HttpMethod.Get, 
  "http://localhost:8091/Api/HelloWorld?bewit=" + bewit);
```

In that way, you can share a link to your Web API with a limited access
for a period of time to someone without having to share any security
credentials.

On the service side is as simple as configuring the HawkMessageHandler
as part of the Web API configuration,

```csharp
var handler = new HawkMessageHandler((id) =>
{
     return new HawkCredential
     {
           Id = id,
           Key = "werxhqb98rpaxn39848xrunpaw3489ruxnpa98w4rxn",
           Algorithm = "hmacsha256",
           User = "steve"
       };
 });

config.MessageHandlers.Add(handler);
```

The handler will automatically detect a bewit token in the query string,
and it will performed all the required validations.

