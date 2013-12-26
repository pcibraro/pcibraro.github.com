---
layout: post
title: "Centralizing Federated Services configuration with SO-Aware"
date: 2010-08-27
comments: true
categories: .NET
---

Configuring a WCF service to use federated authentication in an
organization is not something trivial as it requires some good knowledge
of the available security settings, and more precisely, how to talk to
the existing security token services with the right WCF bindings.

This is something that usually only a few people in the organization
knows how to do it right, so having a way to centralize all this
configuration in a central location and have the rest of the developers
to use becomes really important.    

SO-Aware plays an important role in that sense, allowing the security
experts to configure and store the bindings and behaviors that the
organization will use to secure the services in the service repository.

Developers can later reference, reuse and configure their services and
client applications with those bindings from the repository using a
simple OData API, or the WCF specific classes that SO-Aware also
provides for configuring services and proxies.

A WCF binding for configuring a service with federated authentication
usually looks as follow,

~~~~ {#codeSnippet style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: #f4f4f4; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
<customBinding><binding name="echoClaimsBinding">  <security authenticationMode="IssuedToken"            messageSecurityVersion="WSSecurity11WSTrust13WSSecureConversation13WSSecurityPolicy12BasicSecurityProfile10"            requireSecurityContextCancellation="false">    <issuedTokenParameters tokenType="http://docs.oasis-open.org/wss/oasis-wss-saml-token-profile-1.1#SAMLV2.0">      <claimTypeRequirements>        <add claimType="http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name" isOptional="false"/>        <add claimType="http://SOAwareSamples/2008/05/AgeClaim" isOptional="false"/>              </claimTypeRequirements>      <issuer address="http://localhost:6000/SOAwareSTS"                                               bindingConfiguration="stsBinding"                binding="ws2007HttpBinding">        <identity>          <dns value="WCFSTS"/>        </identity>      </issuer>      <issuerMetadata address="http://localhost:6000/mex"></issuerMetadata>    </issuedTokenParameters>  </security>  <httpTransport/></binding></customBinding>
~~~~

\

You basically have there, the information required by WCF to connect to
the STS (or token issuer), and the claims that the service is expecting.
This binding is also referencing another existing binding “stsBinding”,
that the client will use to connect and secure the communication with
the STS. if you want to store the same thing in SO-Aware, you will need
a way to configure a binding in a way that can reference existing
bindings. That can be done using the “Parent” property as you can see in
the image below,  

[![parentBinding[1]](http://weblogs.asp.net/blogs/cibrax/parentBinding1_566F70B3.png "parentBinding[1]")](http://www.tellagostudios.com/blogEntries/federation/parentBinding.png)

Once you have the binding stored and correctly configured in the the
repository, it’s a matter of using the SO-Aware service host for
configuring existing services with that binding.

~~~~ {#codeSnippet style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: #f4f4f4; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
[ServiceContract()]public interface IEchoClaims{    [OperationContract]    List<string> Echo();}public class EchoClaims : IEchoClaims{    public List<string> Echo()    {        List<string> claims = new List<string>();        IClaimsPrincipal principal = Thread.CurrentPrincipal as IClaimsPrincipal;                foreach (IClaimsIdentity identity in principal.Identities)        {            foreach (Claim claim in identity.Claims)            {                claims.Add(string.Format("{0} - {1}",                    claim.ClaimType, claim.Value));            }        }        return claims;    }}
~~~~

\

~~~~ {#codeSnippet style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: #f4f4f4; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
<serviceRepository url="http://localhost/SoAware/ServiceRepository.svc">     <services>       <service name="ref:EchoClaims(1.0)@dev" type="SOAware.Samples.EchoClaims, Service"/>     </services> </serviceRepository>
~~~~

\
As you can see, the configuration is very straightforward. The developer
configuring the service does not need to know anything about how to
configure the WCF bindings or federated security. He only needs to
reference an existing service configuration in the repository. This
assumes the service was already configured in the Portal or using the
OData API.

 

[![ServiceConfig[1]](http://weblogs.asp.net/blogs/cibrax/ServiceConfig1_0D694EED.png "ServiceConfig[1]")](http://www.tellagostudios.com/blogEntries/federation/ServiceConfig.png)

 

The same thing happens on the client side, no configuration is needed at
all. The developer can use the “ConfigurableProxyFactory” and the
“ConfigurationResolver” classes that SO-Aware provides to automatically
discover and resolve all the service configuration (service address,
bindings and behaviors). In fact, the developer does not know anything
about where the STS is, which binding uses, or which certificates are
used to secure the communication. All that is stored in the repository,
and automatically resolved by the SO-Aware configuration classes.

 

~~~~ {#codeSnippet style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: #f4f4f4; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
static void ExecuteServiceWithMetadataResolution(){    ConfigurableProxyFactory<IEchoClaims> factory = new ConfigurableProxyFactory<IEchoClaims>(        ServiceUri,        "EchoClaims(1.0)",        "dev");    var endpointBehaviors = resolver.ResolveEndpointBehavior("echoClaimsEndpointBehavior");    foreach (var endpointBehavior in endpointBehaviors.Behaviors)    {        if (factory.Endpoint.Behaviors.Contains(endpointBehavior.GetType()))        {            factory.Endpoint.Behaviors.Remove(endpointBehavior.GetType());        }        factory.Endpoint.Behaviors.Add(endpointBehavior);    }    factory.Credentials.UserName.UserName = "joe";    factory.Credentials.UserName.Password = "bar";    IEchoClaims client = factory.CreateProxy();    try    {        string[] claims = client.Echo();        foreach (string claim in claims)        {            Console.WriteLine(claim);        }    }    catch (TimeoutException exception)    {        Console.WriteLine("Got {0}", exception.ToString());        ((IContextChannel)client).Abort();    }    catch (CommunicationException exception)    {        Console.WriteLine("Got {0}", exception.ToString());        IContextChannel channel = (IContextChannel)client;        ((IContextChannel)client).Abort();    }    finally    {        ((IContextChannel)client).Close();    }}
~~~~

\

In addition, as the Secure Token Service could also be implemented with
WCF and WIF, you can also resolve the configuration for that service
from the repository by reusing the “stsBinding” in the given example
(WSTrustServiceContract is one of the service contracts that WIF
provides for implementing a STS).

 

~~~~ {#codeSnippet style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: #f4f4f4; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
<serviceRepository url="http://localhost/SoAware/ServiceRepository.svc">    <services>      <service name="ref:STS(1.0)@dev"         type="Microsoft.IdentityModel.Protocols.WSTrust.WSTrustServiceContract,         Microsoft.IdentityModel, Version=3.5.0.0, Culture=neutral,         PublicKeyToken=31bf3856ad364e35"/>    </services></serviceRepository>
~~~~

\


