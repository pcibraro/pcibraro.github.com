---
layout: post
title: "Using the Katana Authentication handlers with NancyFx"
date: 2013-07-29
comments: true
categories: .NET
---

Once you write an OWIN Middleware service, it can be reused everywhere
as long as OWIN is supported. In my [last
post](http://weblogs.asp.net/cibrax/archive/2013/07/26/writing-an-authenticationhandler-for-katana.aspx),
I discussed how you could write an Authentication Handler in Katana for
Hawk (HMAC Authentication). Good news is NancyFx can be run as an OWIN
handler, so you can use many of existing middleware services, including
the ones that are ship with Katana.

Running NancyFx as a OWIN handler is pretty straightforward, and
discussed in detail as part of the NancyFx documentation
[here](https://github.com/NancyFx/Nancy/wiki/Hosting-nancy-with-owin).
After run the steps described there and you have the application
working, only a few more steps are required to register the additional
middleware services.

The example bellow shows how the Startup class is modified to include
Hawk authentication.

```csharp
public class Startup
{
    public void Configuration(IAppBuilder app)
    {
        app.UseHawkAuthentication(new HawkAuthenticationOptions
        {
            Credentials = (id) =>
            {
                return new HawkCredential
                {
                    Id = "dh37fgj492je",
                    Key = "werxhqb98rpaxn39848xrunpaw3489ruxnpa98w4rxn",
                    Algorithm = "hmacsha256",
                    User = "steve"
                };
            }
        });

        app.UseNancy();
    }
}
```

This code registers the Hawk Authentication Handler on top of the OWIN
pipeline, so it will try to authenticate the calls before the request
messages are passed over to NancyFx.

The authentication handlers in Katana set the user principal in the OWIN
environment using the key “server.User”. The following code shows how
you can get that principal in a NancyFx module,

```csharp
public class HomeModule : NancyModule
{
  public HomeModule()
  {
    Get["/"] = x =>
    {
      var env = (IDictionary<string, object>)Context.Items[NancyOwinHost.RequestEnvironmentKey];

      if (!env.ContainsKey("server.User") || env["server.User"] == null)
      {
          return HttpStatusCode.Unauthorized;
      }

      var identity = (ClaimsPrincipal)env["server.User"];

      return "Hello " + identity.Identity.Name; 
    };
}
```

```csharp
}
```

Thanks to OWIN, you don’t know any details of how these cross cutting
concerns can be implemented in every possible web application framework.

