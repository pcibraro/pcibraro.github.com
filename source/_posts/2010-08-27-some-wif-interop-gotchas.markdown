---
layout: post
title: "Some WIF interop gotchas"
date: 2010-08-27
comments: true
categories: .NET
---

WIF is an excellent framework that allows you to develop an STS in just
a few minutes if you know exactly what you are doing of course :). In my
role as consultant and architect in Tellago, I went through several
projects in which some level of customization was required at wire level
to accomplish some interoperability between a STS built with WIF and
existing federation solutions like ADFS 1.x and OpenSSO.

The idea of this post is to show some of extensibility points that you
will find in WIF to customize the WS-Trust messages, and issued tokens.

**1. Making WIF to speak WS-Trust Feb 2005**

WIF uses by default WS-Trust 1.3, which means that all the generated
WS-Trust messages will use that spec unless you specify a different one.
ADFS 1.x and OpenSSO both uses WS-Trust Feb 2005
([http://schemas.xmlsoap.org/ws/2005/02/trust](http://schemas.xmlsoap.org/ws/2005/02/trust))
for the passive profile to support single sign on over the web.
Therefore, if you want to generate WS-Trust messages that follow that
spec version in your WIF passive STS, you need to modify a little bit
the code you use for processing the RST messages.

~~~~ {#codeSnippet style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
class FederatedPassiveSecurityTokenServiceOperations{    public static void ProcessRequest(HttpRequest request, IPrincipal principal,         SecurityTokenService sts, HttpResponse response,         WSFederationSerializer federationSerializer);    public static SignInResponseMessage ProcessSignInRequest(SignInRequestMessage requestMessage,         IPrincipal principal, SecurityTokenService sts,         WSFederationSerializer federationSerializer);}
~~~~

\

The methods ProcessRequest and ProcessSignRequest in the
FederatedPassiveSecurityTokenServiceOperations class both support an
additional overload for passing the WS-Trust version
(WSFederationSerializer instance).

You can force another WS-Trust version by passing the right serializer
instance. For example, the following code uses the WS-Trust Feb 2005
specification.

~~~~ {#codeSnippet style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
SignInResponseMessage responseMessage = FederatedPassiveSecurityTokenServiceOperations.ProcessSignInRequest(    requestMessage, User, sts,         new WSFederationSerializer(new WSTrustFeb2005RequestSerializer(),             new WSTrustFeb2005ResponseSerializer()));
~~~~

\

**2. Changing the SAML token’s signature algorithms**

WIF uses by default a combination of RSA and SHA 256 for generating the
SAML signature. You can notice this in the generated SAML token,

~~~~ {#codeSnippet style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
<saml:Attribute AttributeName="name" AttributeNamespace="http://schemas.xmlsoap.org/ws/2005/05/identity/claims">    <saml:AttributeValue>MyName</saml:AttributeValue>  </saml:Attribute></saml:AttributeStatement><ds:Signature xmlns:ds="http://www.w3.org/2000/09/xmldsig#">  <ds:SignedInfo>    <ds:CanonicalizationMethod Algorithm="http://www.w3.org/2001/10/xml-exc-c14n#" />    <ds:SignatureMethod Algorithm="http://www.w3.org/2001/04/xmldsig-more#rsa-sha256" />    <ds:Reference URI="#_cecf3c23-824e-4064-846c-b90c03d29700">      <ds:Transforms>        <ds:Transform Algorithm="http://www.w3.org/2000/09/xmldsig#enveloped-signature" />        <ds:Transform Algorithm="http://www.w3.org/2001/10/xml-exc-c14n#" />      </ds:Transforms>      <ds:DigestMethod Algorithm="http://www.w3.org/2001/04/xmlenc#sha256" />      <ds:DigestValue>1flG08Axm71C0isY2wLR0C9jqgfIebNoG2nlIO+jO+s=</ds:DigestValue>    </ds:Reference> </ds:SignedInfo>
~~~~

\
ADFS 1.x and OpenSSO use a combination of RSA and SHA, so that’s also
something else you need to customize. The “X509SigningCredentials”
instance that you pass in the constructor of the Secure token service
configuration also contains an overload to change the signature
algorithm. The following code creates a new instance of
“X509SigningCredentials” that uses SHA rather than SHA 256.

 

~~~~ {#codeSnippet style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
new X509SigningCredentials(    CertificateUtil.GetCertificate(StoreName.TrustedPeople,         StoreLocation.LocalMachine,         "CN=Test"),     "http://www.w3.org/2000/09/xmldsig#rsa-sha1",     "http://www.w3.org/2000/09/xmldsig#sha1"))
~~~~

\

**3. Adding an authentication statement to the issued SAML token**

 

This part is very tricky, as WIF does not add by default an
authentication statement in the SAML token unless you use an specific
claim type (NameIdentifier) with some custom properties.

 

~~~~ {#codeSnippet style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
Claim nameIdentifier = new Claim(System.IdentityModel.Claims.ClaimTypes.NameIdentifier,     "foo@test.com");nameIdentifier.Properties["http://schemas.xmlsoap.org/ws/2005/05/identity/claimproperties/format"]     = "http://schemas.xmlsoap.org/claims/UPN";outputIdentity.Claims.Add(nameIdentifier);outputIdentity.Claims.Add(new Claim(ClaimTypes.AuthenticationMethod, "http://microsoft/geneva"));outputIdentity.Claims.Add(new Claim(ClaimTypes.AuthenticationInstant, XmlConvert.ToString(DateTime.Now, XmlDateTimeSerializationMode.Utc)));
~~~~

\

The code above will generate an authentication statement like this in
the SAML token, which is something equivalent to what ADFS 1.x or
OpenSSO would generate.

~~~~ {#codeSnippet style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
<saml:AttributeStatement>  <saml:Subject>    <saml:NameIdentifier Format="http://schemas.xmlsoap.org/claims/UPN">        foo@test.com    </saml:NameIdentifier>    <saml:SubjectConfirmation>      <saml:ConfirmationMethod>            urn:oasis:names:tc:SAML:1.0:cm:bearer        </saml:ConfirmationMethod>    </saml:SubjectConfirmation>  </saml:Subject>    </saml:AttributeStatement>
~~~~

\

 

