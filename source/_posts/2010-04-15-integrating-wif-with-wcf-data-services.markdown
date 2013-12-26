---
layout: post
title: "Integrating WIF with WCF Data Services"
date: 2010-04-15
comments: true
categories: .NET
---

A time ago I
[discussed](http://weblogs.asp.net/cibrax/archive/2009/03.aspx) how a
custom REST Starter kit interceptor could be used to parse a SAML token
in the Http Authorization header and wrap that into a ClaimsPrincipal
that the WCF services could use. The thing is that code was initially
created for Geneva framework, so it got deprecated quickly. I recently
needed that piece of code for one of projects where I am currently
working on so I decided to update it for WIF. As this interceptor can be
injected in any host for WCF REST services, also represents an excellent
solution for integrating claim-based security into WCF Data Services
(previously known as ADO.NET Data Services).

The interceptor basically expects a SAML token in the Authorization
header. If a token is found, it is parsed and a new ClaimsPrincipal is
initialized and injected in the WCF authorization context.

public class SamlAuthenticationInterceptor : RequestInterceptor \
{ \
  SecurityTokenHandlerCollection handlers;

  public SamlAuthenticationInterceptor() \
    : base(false) \
  { \
    this.handlers =
FederatedAuthentication.ServiceConfiguration.SecurityTokenHandlers; \
  }

  public override void ProcessRequest(ref RequestContext requestContext)
\
  { \
    SecurityToken token =
ExtractCredentials(requestContext.RequestMessage);

    if (token != null) \
    { \
      ClaimsIdentityCollection claims = handlers.ValidateToken(token);

      var principal = new ClaimsPrincipal(claims); \
      InitializeSecurityContext(requestContext.RequestMessage,
principal); \
    } \
    else \
    { \
      DenyAccess(ref requestContext); \
    } \
  }

  private void DenyAccess(ref RequestContext requestContext) \
  { \
    Message reply = Message.CreateMessage(MessageVersion.None, null); \
    HttpResponseMessageProperty responseProperty = new
HttpResponseMessageProperty() { StatusCode = HttpStatusCode.Unauthorized
};

    responseProperty.Headers.Add("WWW-Authenticate", \
          String.Format("Basic realm=\\"{0}\\"", ""));

    reply.Properties[HttpResponseMessageProperty.Name] =
responseProperty; \
    requestContext.Reply(reply);

    requestContext = null; \
  }

  private SecurityToken ExtractCredentials(Message requestMessage) \
  { \
    HttpRequestMessageProperty request = (HttpRequestMessageProperty) 
requestMessage.Properties[HttpRequestMessageProperty.Name];

    string authHeader = request.Headers["Authorization"];

    if (authHeader != null && authHeader.Contains("\<saml")) \
    { \
      XmlTextReader xmlReader = new XmlTextReader(new
StringReader(authHeader));

      var col =
SecurityTokenHandlerCollection.CreateDefaultSecurityTokenHandlerCollection();
\
      SecurityToken token = col.ReadToken(xmlReader); \
                                 \
      return token; \
    }

    return null; \
  }

  private void InitializeSecurityContext(Message request, IPrincipal
principal) \
  { \
    List\<IAuthorizationPolicy\> policies = new
List\<IAuthorizationPolicy\>(); \
    policies.Add(new PrincipalAuthorizationPolicy(principal)); \
    ServiceSecurityContext securityContext = new
ServiceSecurityContext(policies.AsReadOnly());

    if (request.Properties.Security != null) \
    { \
      request.Properties.Security.ServiceSecurityContext =
securityContext; \
    } \
    else \
    { \
      request.Properties.Security = new SecurityMessageProperty() {
ServiceSecurityContext = securityContext }; \
     } \
   }

   class PrincipalAuthorizationPolicy : IAuthorizationPolicy \
   { \
     string id = Guid.NewGuid().ToString(); \
     IPrincipal user;

     public PrincipalAuthorizationPolicy(IPrincipal user) \
     { \
       this.user = user; \
     }

     public ClaimSet Issuer \
     { \
       get { return ClaimSet.System; } \
     }

     public string Id \
     { \
       get { return this.id; } \
     }

     public bool Evaluate(EvaluationContext evaluationContext, ref
object state) \
     { \
       evaluationContext.AddClaimSet(this, new
DefaultClaimSet(System.IdentityModel.Claims.Claim.CreateNameClaim(user.Identity.Name)));
\
       evaluationContext.Properties["Identities"] = new
List\<IIdentity\>(new IIdentity[] { user.Identity }); \
       evaluationContext.Properties["Principal"] = user; \
       return true; \
     } \
   } \

A WCF Data Service, as any other WCF Service, contains a service host
where this interceptor can be injected. The following code illustrates
how that can be done in the “svc” file.

\<%@ ServiceHost Language="C\#" Debug="true"
Service="ContactsDataService" \
                Factory="AppServiceHostFactory" %\>

using System; \
 using System.ServiceModel; \
 using System.ServiceModel.Activation; \
 using Microsoft.ServiceModel.Web;

class AppServiceHostFactory : ServiceHostFactory \
 { \
   protected override ServiceHost CreateServiceHost(Type serviceType,
Uri[] baseAddresses) \
  { \
    WebServiceHost2 result = new WebServiceHost2(serviceType, true,
baseAddresses);

    result.Interceptors.Add(new SamlAuthenticationInterceptor()); \
            \
    return result; \
  } \
}

WCF Data Services includes an specific WCF host of out the box
(DataServiceHost). However, the service is not affected at all if you
replace it with a custom one as I am doing in the code above
(WebServiceHost2 is part of the REST Starter kit).

Finally, the client application needs to pass the SAML token somehow to
the data service. In case you are using any Http client library for
consuming the data service, that’s easy to do, you only need to include
the SAML token as part of the “Authorization” header. If you are using
the auto-generated data service proxy, a little piece of code is needed
to inject a SAML token into the DataServiceContext instance. That class
provides an event “SendingRequest” that any client application can
leverage to include custom code that modified the Http request before it
is sent to the service. So, you can easily create an extension method
for the DataServiceContext that negotiates the SAML token with an
existing STS, and adds that token as part of the “Authorization” header.

public static class DataServiceContextExtensions \
 {     

  public static void ConfigureFederatedCredentials(this
DataServiceContext context, string baseStsAddress, string realm) \
  { \
    string address = string.Format(STSAddressFormat, baseStsAddress,
realm); \
             \
    string token = NegotiateSecurityToken(address); \
    context.SendingRequest += (source, args) =\> \
    { \
      args.RequestHeaders.Add("Authorization", token); \
    }; \
  }

private string NegotiateSecurityToken(string address) \
 { \
 }

}

I left the NegociateSecurityToken method empty for this extension as it
depends pretty much on how you are negotiating tokens from an existing
STS. In case you want to end-to-end REST solution that involves an Http
endpoint for the STS, you should definitely take a look at the
[Thinktecture starter STS project](http://startersts.codeplex.com/) in
codeplex.

