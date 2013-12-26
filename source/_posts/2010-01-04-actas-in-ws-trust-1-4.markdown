---
layout: post
title: "ActAs in WS-Trust 1.4"
date: 2010-01-04
comments: true
categories: .NET
---

WS-Trust 1.4 introduced a new feature called as “ActAs” for addressing
common scenarios where an application needs to call a service on behalf
of the logged user or a service needs to call another service on behalf
of the original caller. These are typical examples of what is usually
resolved with the “[Trusted Subsystem”
pattern](http://www.soapatterns.org/trusted_subsystem.php).

“ActAs” is not more than a new element in the RST message for including
additional information about the original caller when a token is
negotiated with the STS for consuming the final service in the Relying
Party (that means that a trust relationship already exists between the
STS and the final service). That element usually takes the form of a
token with identity claims, which are secured (encrypted/signed) and
included together with the credentials of the client application that is
negotiating the token with the STS.

As this element is part of the WS-Trust specification, it only makes
sense in scenarios where the client authentication is delegated to an
STS. I already discussed this scenario several times in the past. The
following message shows how this element is included in a RST message,

\<trust:RequestSecurityToken
xmlns:trust="[http://docs.oasis-open.org/ws-sx/ws-trust/200512](http://docs.oasis-open.org/ws-sx/ws-trust/200512)"\>
\
  \<tr:ActAs
xmlns:tr="[http://docs.oasis-open.org/ws-sx/ws-trust/200802](http://docs.oasis-open.org/ws-sx/ws-trust/200802)"\>
\
    \<saml:Assertion MajorVersion="1" MinorVersion="1"
AssertionID="\_cf5f224e-4bae-4f2d-9800-248023ac0e4d" Issuer="PassiveSTS"
IssueInstant="2010-01-04T15:20:14.506Z"
xmlns:saml="urn:oasis:names:tc:SAML:1.0:assertion"\> \
      \<saml:Conditions NotBefore="2010-01-04T15:20:14.470Z"
NotOnOrAfter="2010-01-04T16:20:14.470Z"\>\</saml:Conditions\> \
        \<saml:AttributeStatement\> \
          \<saml:Subject\> \
            \<saml:NameIdentifier
Format="[uid:0@stonehenge.comhttp://schemas.xmlsoap.org/claims/UPN"\>uid:0@stonehenge.com\</saml:NameIdentifier](http://schemas.xmlsoap.org/claims/UPN%22%3Euid:0@stonehenge.com%3C/saml:NameIdentifier)\>
\
            \<saml:SubjectConfirmation\> \
             
\<saml:ConfirmationMethod\>urn:oasis:names:tc:SAML:1.0:cm:bearer\</saml:ConfirmationMethod\>
\
            \</saml:SubjectConfirmation\> \
          \</saml:Subject\> \
          \<saml:Attribute AttributeName="role"
AttributeNamespace="<http://microsoft>"\> \
            \<saml:AttributeValue\>staff\</saml:AttributeValue\>\\ \
          \</saml:Attribute\> \
        \</saml:AttributeStatement\> \
        \<saml:AuthenticationStatement
AuthenticationMethod="<http://microsoft/geneva>"
AuthenticationInstant="2010-01-04T15:20:14.481Z"\> \
          \<saml:Subject\> \
            \<saml:NameIdentifier
Format="[uid:0@stonehenge.comhttp://schemas.xmlsoap.org/claims/UPN"\>uid:0@stonehenge.com\</saml:NameIdentifier](http://schemas.xmlsoap.org/claims/UPN%22%3Euid:0@stonehenge.com%3C/saml:NameIdentifier)\>
\
            \<saml:SubjectConfirmation\> \
             
\<saml:ConfirmationMethod\>urn:oasis:names:tc:SAML:1.0:cm:bearer\</saml:ConfirmationMethod\>
\
            \</saml:SubjectConfirmation\> \
          \</saml:Subject\> \
        \</saml:AuthenticationStatement\> \
        \<ds:Signature
xmlns:ds="[http://www.w3.org/2000/09/xmldsig](http://www.w3.org/2000/09/xmldsig)\#"\>
\
        \</ds:Signature\> \
      \</saml:Assertion\> \
    \</tr:ActAs\> \
 
\<trust:ComputedKeyAlgorithm\>http://docs.oasis-open.org/ws-sx/ws-trust/200512/CK/PSHA1\</trust:ComputedKeyAlgorithm\>
\
\</trust:RequestSecurityToken\>

If you still want to support an scenario like this when an STS is not
involved for client authentication, the closer thing you can find is the
client authentication through supporting tokens method. I described this
scenario in this
[post](http://weblogs.asp.net/cibrax/archive/2008/01/22/authenticating-users-with-supporting-tokens-in-wcf.aspx),
and Dominick did a better job giving a [concrete
implementation](http://www.leastprivilege.com/UserNameSupportingTokenInWCF.aspx)
in WCF that uses an Username Token as supporting token, a also a [SAML
token](http://www.leastprivilege.com/UsingSAMLAsAClientCredentialTypeInWCFWithGeneva.aspx),
which represents something very useful.

[![WSTrust1](http://weblogs.asp.net/blogs/cibrax/WSTrust1_thumb_579714EF.jpg "WSTrust1")](http://weblogs.asp.net/blogs/cibrax/WSTrust1_6E19D943.jpg)

[![WSTrust2](http://weblogs.asp.net/blogs/cibrax/WSTrust2_thumb_14D7C9B7.jpg "WSTrust2")](http://weblogs.asp.net/blogs/cibrax/WSTrust2_766D88CD.jpg)

The images illustrate two scenarios where this new feature makes a lot
of sense. Let’s discuss both of them more in detail.

Scenario 1: A Service A calling a Service B

In this scenario, we have a Service A that needs to make a call to an
operation in the Service B. Both services are expecting an token issued
by an STS that they trust (The example assumes that they both trust the
same STS). This is how “ActAs” works in this scenario.

​1. The client negotiates a SAML token with the STS for consuming the
service A

​2. The client invokes an operation in the Service A using the SAML
token it got from the STS as client credentials.

​3. Now, the Service A needs to invoke an operation in the Service B so
it has to negotiate a new SAML token from the STS first. In order to do
that, it includes the SAML token sent by the client as part of the ActAs
element, and secures the message using any of the traditional
WS-Security profiles. For example, Mutual Certificate, a client
certificate for authenticating the client (Service A) and a service
certificate for authenticating the service and protecting the messages.
The STS issues a new token for consuming the service B using all the
information received in the RST message.

​4. The service A invokes an operation in the Service B using the SAML
token it just got from the STS as client credentials.

Scenario 2: A Web Application calling a Service

In this scenario, the Web Application authenticates all its user through
an STS that implements the WS-Trust passive profile. This means that the
web application is a claim-aware application that receives the user
identity as claims from a token sent by the STS (The user first
authenticates in the STS, step 1). As the web application already has a
token with the user claims, which is the token it received from the
passive STS, it can include it as the ActAs element when it negotiates a
SAML token from the Active STS (step 2). Once the application gets a
SAML token from the Active STS, it can actually use it for consuming the
service (step 3)

Fortunately, WIF already includes support in the client API for
negotiating a token with the ActAs element, and also support for parsing
and getting access to that information in the STS implementation. 
What’s more, the implementation of this new feature in WIF is totally
interoperable with other Service stacks as well. This is something that
have been proved in the Apache Stonehenge project by proving an example
application that shows different interoperability scenarios with
claim-based security with other service stacks like Sun Metro or WOS2.

 

 

On the client side, WIF provides a couple of extensions methods in the
WCF channel for including a token in the ActAs element.

public static T CreateChannelActingAs\<T\>(this ChannelFactory\<T\>
factory, SecurityToken actAs); \
public static T CreateChannelActingAs\<T\>(this ChannelFactory\<T\>
factory, EndpointAddress address, SecurityToken actAs); \
public static T CreateChannelActingAs\<T\>(this ChannelFactory\<T\>
factory, EndpointAddress address, Uri via, SecurityToken actAs);

In case you are using a SAML token, you need to negotiate it out of band
with the WSTrustChannel class or use the one that is available in the
web application when passive authentication is used with an STS (and the
setting saveBootstapTokens is enabled in the microsoft.IdentityModel
section). The following code illustrates a sample that uses the SAML
token available as part of the Bootstrap tokens in the IClaimsPrincipal
instance attached to the current thread.

SecurityToken callerToken = null;

IClaimsPrincipal claimsPrincipal = Thread.CurrentPrincipal as
IClaimsPrincipal; \
 if (claimsPrincipal != null) \
 { \
   foreach (IClaimsIdentity claimsIdentity in
claimsPrincipal.Identities) \
   { \
     if (claimsIdentity.BootstrapToken is SamlSecurityToken) \
     { \
       callerToken = claimsIdentity.BootstrapToken; \
       break; \
     } \
  } \
 }

var channel = channelFactory.CreateChannelActingAs(callerToken);

The STS implementation can get the ActAs element information from the
RequestSecurityToken.ActAs property,

if (request.ActAs != null) \
{ \
  IClaimsIdentity actAsIdentity = new ClaimsIdentity(); \
  CopyClaims(request.ActAs.GetSubject().First(), actAsIdentity);

  // Find the last delegate in the actAs identity \
  IClaimsIdentity lastActingVia = actAsIdentity; \
  while (lastActingVia.Actor != null) \
  { \
    lastActingVia = lastActingVia.Actor; \
  }

  // Put the caller's identity as the last delegate to the ActAs
identity \
  lastActingVia.Actor = outputIdentity;

  // Return the actAsIdentity instead of the caller's identity in this
case \
  outputIdentity = actAsIdentity; \
}

The apache stonehenge application or the [Fabrikam shipping
sample](http://code.msdn.microsoft.com/FabrikamShipping) in codeplex are
using this feature, so you might be interested in taking a look there.

