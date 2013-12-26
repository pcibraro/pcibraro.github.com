---
layout: post
title: "Brokered authentication for REST active clients with SAML"
date: 2009-03-06
comments: true
categories: Federation
---

I have been thinking for a while about what could be a good way to
support brokered authentication for active REST clients. Something I did
not want to do was to force the use of WS-Trust Active profile, which is
in essence SOAP based.

Some of the qualities attributes that are easy to reach with REST
services, such as simplicity, interoperability and scalability can
definitely be affected with the introduction of a additional SOAP stack
for negotiating an identity token. WS-Trust passive requestor profile,
on the other hand, was designed for dumb clients like web browsers,
clients that do not have capabilities to handle cryptographic materials
or the SOAP stack itself.  This profile basically hides most of the
WS-Trust details from client applications through a sequence of http
redirections, which could be helpful in this scenario for negotiating a
token and still keep simple REST clients. However, as some user
interaction is required, this profile is not suitable for consuming REST
services from desktop applications or other active client applications.

If we take a deep look at the functionality provided by a Secure Token
Service (STS), it is not more than a service that handle the lifecycle
of a identity token, it knows how to issue a token, renew it or finally
cancel it when it is not longer need it.  If we see all these scenarios
from a point of view of REST, an identity token is just a resource,
something that can be created, updated or even deleted. Of course, there
is not any spec available yet for this scenario, all I will show here is
just an possible implementation of a Restful STS.

The mapping of supported Ws-Trust actions to http verbs for my Restful
STS is defined below,

-   Issue = POST, creates or issues a new token resource (A SAML token)
-   Renew = PUT, renew an existing token
-   Cancel = DELETE, cancel an existing token
-   GET, gets an existing token (There is not such thing in Ws-Trust)

I leave out the "Validate" action as part of this implementation.

What I have created for this example is a REST facade layered on top of
a STS implementation with the Geneva Framework. The definition of
service contract for this Restful STS for supporting that mapping should
look like this,

~~~~ {.c# name="code"}
[ServiceContract]public interface IRestSts{    [OperationContract]    [WebInvoke(UriTemplate="Tokens", Method="POST", RequestFormat=WebMessageFormat.Xml, ResponseFormat=WebMessageFormat.Xml)]    RequestSecurityTokenResponse IssueToken(RequestSecurityToken request);     [OperationContract]    [WebInvoke(Method = "PUT", UriTemplate = "Tokens/{tokenId}", RequestFormat = WebMessageFormat.Xml, ResponseFormat = WebMessageFormat.Xml)]    RequestSecurityTokenResponse RenewToken(string tokenId);     [OperationContract]    [WebInvoke(Method = "DELETE", UriTemplate = "Tokens/{tokenId}", RequestFormat = WebMessageFormat.Xml, ResponseFormat = WebMessageFormat.Xml)]    void CancelToken(string tokenId);     [OperationContract]    [WebGet(UriTemplate = "Tokens/{tokenId}", RequestFormat = WebMessageFormat.Xml, ResponseFormat = WebMessageFormat.Xml)]    RequestSecurityTokenResponse GetToken(string tokenId);    }
~~~~

As I mentioned before, the client has to first acquire a token from the
STS, that can be done with a regular Http POST containing a
RequestSecurityToken message.

[![Issue\_REST](/images/legacy/FederatedidentityinRESTservices_A72D/Issue_REST_thumb.gif)](/images/legacy/FederatedidentityinRESTservices_A72D/Issue_REST.gif)

The message embedded in the request body to the STS looks like this,

~~~~ {.xml name="code"}
<RequestSecurityToken xmlns="http://schemas.xmlsoap.org/ws/2005/02/trust">
    <AppliesTo>https://localhost/MyService</AppliesTo>
    <TokenType>http://docs.oasis-open.org/wss/oasis-wss-saml-token-profile-1.1#SAMLV1.1</TokenType>
</RequestSecurityToken>
~~~~

And the corresponding response like this,

~~~~ {.xml name="code"}
<RequestSecurityTokenResponse xmlns="http://schemas.xmlsoap.org/ws/2005/02/trust" xmlns:i="http://www.w3.org/2001/XMLSchema-instance">
    <Links>
        <Link>
            <href>http://localhost:7362/STSWindows/Service.svc/_8a6fc87b-7e6a-45c9-a479-20ea42113e40</href>
            <rel>self</rel>
            <type>application/xml</type>
        </Link>
    </Links>
    <RequestedSecurityToken>....</RequestedSecurityToken>
    <TokenType>http://docs.oasis-open.org/wss/oasis-wss-saml-token-profile-1.1#SAMLV1.1</TokenType>
</RequestSecurityTokenResponse>
~~~~

Both calls, the first one to get the token from the STS, and the second
call to invoke the service in the Relying party should be protected with
transport security to avoid any middle in the man attack.

In this sample, the STS is using basic authentication to authenticate
the user trying to get access to the token. If the authentication
succeed, the STS implemented with Geneva will provide the necessary
claims associated with that user.

The code on the client side to ask for a new token is quite simple,

static string GetToken(string address, string appliesTo, string
username, string password)

{

    RequestSecurityToken request = new RequestSecurityToken

    {

        TokenType =
"http://docs.oasis-open.org/wss/oasis-wss-saml-token-profile-1.1\#SAMLV1.1",

        AppliesTo = appliesTo

    };

    DataContractSerializer requestSerializer = new
DataContractSerializer(typeof(RequestSecurityToken));

    WebRequest webRequest = HttpWebRequest.Create(address);

    webRequest.Method = "POST";

    webRequest.ContentType = "application/xml";

    webRequest.Credentials = new NetworkCredential(username, password);

    using (var st = webRequest.GetRequestStream())

    {

        requestSerializer.WriteObject(st, request);

        st.Flush();

    }

    WebResponse webResponse = webRequest.GetResponse();

    DataContractSerializer responseSerializer = new
DataContractSerializer(typeof(RequestSecurityTokenResponse));

    using (var st = webResponse.GetResponseStream())

    {

        var response =
(RequestSecurityTokenResponse)responseSerializer.ReadObject(st);

        return response.RequestedSecurityToken;

    }

}

It creates a new RequestSecurityToken message, provides the user
credentials and post that information to the STS. The response from the
STS is a RequestSecurityTokenResponse containing the issued token,
that's what this method returns in response.RequestedSecurityToken.

Once the client gets the issued token from the response, it can include
it as part of the request message to the relying party's service. For
this sample, I decided to include the token in the "Authorization"
header, which is a common mechanism to attach authentication credentials
in a request message to a REST service (Basic authentication, and other
authentication mechanisms use the same approach).

WebRequest webRequest = HttpWebRequest.Create(address);

webRequest.Method = "GET";

webRequest.Headers["Authorization"] = token;

Now, the hard part, the Relying Party needs a way to parse the token and
authenticate the user before calling the service implementation.
Fortunately, the guys from the WCF REST Starter kit have provide an
excellent solution for this kind of scenarios, message interceptors.
What I did here was to implement a message interceptor for SAML tokens,
which internally used the Geneva Framework for performing all the
validations and parsing the token.  An easy way to inject message
interceptors in a service implementation is through a custom service
factory (Zero config deployment),

class AppServiceHostFactory : ServiceHostFactory

{

    protected override ServiceHost CreateServiceHost(Type serviceType,
Uri[] baseAddresses)

    {

        WebServiceHost2 result = new WebServiceHost2(serviceType, true,
baseAddresses);

        result.Interceptors.Add(new
MessageInterceptors.SamlAuthenticationInterceptor(new
TrustedIssuerNameRegistry()));

        return result;

    }

}

The "TrustedIssuerNameRegistry" is a just a simple implementation of a
Geneva "IssuerNameRegistry" provider that validates the issuer of the
SAML token.

All this stuff is of course transparent to the service implementation,
it only receives a bunch of claims representing the user identity. Those
claims can be got accessed through the current user principal. In the
code below, the service generates a feed with all the received claims.

IClaimsIdentity identity =
(IClaimsIdentity)Thread.CurrentPrincipal.Identity;

var feed = new SyndicationFeed()

{

    Id = "http://Claims",

    Title = new TextSyndicationContent("My claims"),

};

feed.Items = identity.Claims.Select(c =\>

    new SyndicationItem()

    {

        Id = Guid.NewGuid().ToString(),

        Title = new TextSyndicationContent(c.ClaimType),

        LastUpdatedTime = DateTime.UtcNow,

        Authors =

            {

                new SyndicationPerson()

                {

                    Name = c.Issuer

                }

            },

        Content = new TextSyndicationContent(c.Value)

    }

);

The complete sample is available to download from
[here](/images/legacy/FederatedRest.zip). Note, it uses
the latest Geneva Framework bits (And also the X509 certificates
included with the samples, just run the certificate setup file included
with the framework).

