---
layout: post
title: "Writing an AuthenticationHandler for Katana"
date: 2013-07-26
comments: true
categories: .NET
---

As I discussed in my previous
[post](http://weblogs.asp.net/cibrax/archive/2013/07/22/getting-started-with-owin-and-katana.aspx),
Katana is pretty much organized in middleware services.  One of those
middleware services is authentication, which provides some built-in
implementations for existing OAuth providers such as Facebook, Twitter,
Google or Microsoft, and also an implementation for Forms authentication
with cookies.  All those implementations are currently distributed as
Nuget packages under the name of Microsoft.Owin.Security.\*, where the
last part identifies the name of the implementation (e.g.
Microsoft.Owin.Security.Twitter).

Microsoft.Owin.Security is also the core project where you can find the
base classes for writing a new authentication handler, and the ones that
all the these implementations use.

At first glance, the core class that you will use to create a new
authentication handler is
Microsoft.Owin.Security.Infrastructure.AuthenticationHandler\<T\>, where
T is class that derives from AuthenticationOptions and contains all the
properties for initializing the handler.

AuthenticationHandler\<T\> derives from
Microsoft.Owin.Security.Infrastructure.AuthenticationHandler, which
provides the following definition.

```csharp
public abstract class AuthenticationHandler
{
   protected OwinRequest Request;
   protected OwinResponse Response;

   protected AuthenticationHandler();

   protected virtual Task ApplyResponseChallenge();

   protected virtual Task ApplyResponseCore();

   protected virtual Task ApplyResponseGrant();

   protected abstract Task<AuthenticationTicket> AuthenticateCore();
}
```

This class basically provides access to the current request/response
OWIN messages, and also a few methods that can be overridden as part of
the implementation.

-   ApplyResponseChallenge: This method can be overridden to send a
    authentication challenge when a middleware service in the upper
    layer denies the execution (e.g 401 status code). For example, this
    can be useful for basic authentication.
-   ApplyResponseCore: This method by default calls
    ApplyResponseChallenge and ApplyResponseGrant. It can be changed to
    make some additional processing of the Response message.
-   ApplyResponseGrant: This method is only useful for authentication
    methods that needs to implement sign-in/sign-out concerns such as
    OAuth or Forms authentication. You won’t be using this for some
    authentication options such as basic or hmac authentication.
-   AuthenticateCore: This is the more important method, and the one
    where all the main implementation of the authentication handler
    lives. The AuthenticationTicket will contain the identity of the
    authenticated user or null if the user couldn’t be authenticated.

The AuthenticationOptions base class only contains the following
structure,

```csharp
public abstract class AuthenticationOptions
{
  protected AuthenticationOptions(string authenticationType);

  public AuthenticationMode AuthenticationMode { get; set; }

  public string AuthenticationType { get; set; }

  public AuthenticationDescription Description { get; set; }
}
```

-   AuthenticationMode. It’s an enumeration with two possible values
    Active/Passive. It basically identifies the http flow for the
    authentication implementation. Some authentication methods like
    Basic or HMac are Active, so it can be used directly against the
    current request without requiring additional http redirections. Some
    other methods like OAuth or Forms are not, so those have to
    identified as Passive. This setting will affect the authentication
    middleware behaves, so you have to set the correct value for it.
-   AuthenticatinType. It represents the authentication scheme. For
    example, Basic.
-   AuthenticationDescription. It can be used to provide more
    information about the authentication method.

Only these two classes are used so far to create a new authentication
handler. However, you will also need a another class that acts as a
factory, and it is used to inject the handler into the OWIN pipeline.
That class is
Microsoft.Owin.Security.Infrastructure.AuthenticationMiddleware.

```csharp
public abstract class AuthenticationMiddleware<TOptions> : OwinMiddleware where TOptions : Microsoft.Owin.Security.AuthenticationOptions
{
  public AuthenticationMiddleware(OwinMiddleware next, TOptions options);

  public TOptions Options { get; set; }

  protected abstract AuthenticationHandler<TOptions> CreateHandler();
}
```

This class basically receives the Options that must be passed to the
AuthenticationHandler, and also contains a method CreateHandler where
the handler is actually created.

I will use the rest of this post to describe the implementation of a
handler for doing HMac authentication using Hawk.

The first step in the implementation is to create a custom
AuthenticationOptions class.

```csharp
public class HawkAuthenticationOptions : AuthenticationOptions
{
  public const string Scheme = "Hawk";

  public HawkAuthenticationOptions()
       : base(Scheme)
  {
   }

  public Func<string, HawkCredential> Credentials { get; set; }
}
```

The Credentials property is a callback that the authentication handler
implementation will use to resolve the user/key to verify the HMAC
received in the authorization header. In a Basic Authentication
implementation, you could potentially have a callback or a reference to
a repository to verify the username and password received in the
authorization header. I am also passing “Hawk” as part of the
constructor to set the authentication scheme associated to the handler.

The next step is to implement the AuthenticationHandler.

```csharp
public class HawkAuthenticationHandler : AuthenticationHandler<HawkAuthenticationOptions>
{
  private readonly ILogger logger;

  public HawkAuthenticationHandler(ILogger logger)
  {
      this.logger = logger;
  }

  protected override Task<AuthenticationTicket> AuthenticateCore()
  {
     AuthenticationHeaderValue authorization = null;

     if (Request.GetHeader("authorization") != null)
     {
         authorization = AuthenticationHeaderValue.Parse(Request.GetHeader("authorization"));
      }

     if (authorization != null &&
          !string.Equals(authorization.Scheme, HawkAuthenticationOptions.Scheme))
     {
          this.logger.WriteInformation(string.Format("Authorization skipped. Schema found {0}",
               authorization.Scheme));

          return Task.FromResult(EmptyTicket());
      }

      if (authorization == null ||
         string.IsNullOrWhiteSpace(authorization.Scheme))
      {
          this.logger.WriteWarning("Authorization header not found");

          return Task.FromResult(EmptyTicket());
      }
      else
      {
          if (string.IsNullOrWhiteSpace(authorization.Parameter))
          {
              this.logger.WriteWarning("Invalid header format");
                    
              return Task.FromResult(EmptyTicket());
          }

         if (string.IsNullOrWhiteSpace(Request.Host))
         {
              this.logger.WriteWarning("Missing Host header");
                    
              return Task.FromResult(EmptyTicket());
          }

          try
          {
              var principal = Hawk.Authenticate(authorization.Parameter,
                   Request.Host,
                   Request.Method,
                   Request.Uri,
                   this.Options.Credentials);

              var identity = (ClaimsIdentity)((ClaimsPrincipal)principal).Identity;
              var ticket = new AuthenticationTicket(identity, (AuthenticationExtra)null);

              return Task.FromResult(ticket);
           }
           catch (SecurityException ex)
           {
                this.logger.WriteWarning(ex.Message);

                return Task.FromResult(EmptyTicket());
           }
        }
    }

    protected override Task ApplyResponseChallenge()
    {
         if (Response.StatusCode != 401)
         {
              return Task.FromResult<object>(null);
          }

         var ts = Hawk.ConvertToUnixTimestamp(DateTime.Now).ToString();
         var challenge = string.Format("ts=\"{0}\" ntp=\"{1}\"",
                ts, "pool.ntp.org");

         Response.AddHeader("WWW-Authenticate", HawkAuthenticationOptions.Scheme + " " + challenge);
         
        return Task.FromResult<object>(null);
     }

     private static AuthenticationTicket EmptyTicket()
     {
         return new AuthenticationTicket(null, (AuthenticationExtra)null);
      }
 }
```

The implementation is Active, so only overrides the AuthenticationCore
and ApplyResponseChallenge methods. The AuthenticationCore
implementation tries to locate a Authorization Header with a format that
follows the Hawk specification to authenticate the call. If an
Authorization header is not found or it can not be parsed, an
AuthenticationTicket instance with an empty identity is returned.
Otherwise, the identity is set in the ticket and returned as part of the
task. This implementation also uses the logging facilities provided by
Katana.

The last step is to implement the AuthenticationMiddleware class.

```csharp
public class HawkAuthenticationMiddleware : AuthenticationMiddleware<HawkAuthenticationOptions>
{
    private readonly ILogger logger;

    public HawkAuthenticationMiddleware(
        OwinMiddleware next,
        IAppBuilder app,
        HawkAuthenticationOptions options) : base(next, options)
    {
         this.logger = app.CreateLogger<HawkAuthenticationHandler>();
     }

     protected override AuthenticationHandler<HawkAuthenticationOptions> CreateHandler()
     {
         return new HawkAuthenticationHandler(this.logger);
     }
 }
```

This implementation only overrides the CreateHandler method to return a
new HawkAuthenticationHandler instance. As that instance relies on the
Katana logger, the logger instance is first created from the IAppBuilder
instance injected in the constructor.

Once you have the AuthenticationMiddleware implementation completed, you
will want to inject it in the OWIN pipeline to use it in an existing
application. An extension method can be provided to make this task
easier for the developer.

```csharp
public static class HawkAuthenticationExtensions
{
    public static IAppBuilder UseHawkAuthentication(this IAppBuilder app, HawkAuthenticationOptions options)
    {
        app.Use(typeof(HawkAuthenticationMiddleware), app, options);
        app.UseStageMarkerAuthenticate();
        return app;
     }
 }
```

The following example shows how this extension is used to register the
HawkAuthenticationMiddleware service in an application that uses Web API
with OWIN

```csharp
public class Startup
{
     public void Configuration(IAppBuilder app)
     {
         app.SetLoggerFactory(new ConsoleLoggerFactory());

         var config = new HttpConfiguration();
         config.Routes.MapHttpRoute("Default", "api/{controller}");
            
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
 
        app.UseWebApi(config);
            
     }
 }
```

All the code provided as part of this example for Hawk can be found in
the [HawkNet github project](https://github.com/pcibraro/hawknet).

