---
layout: post
title: "Single Sign-On scenarios with Federation"
date: 2008-02-11
comments: true
categories: WCF
---

Before reading this post, if you know the basic concepts and ideas
behind the implementation of an STS with WCF, go ahead and jump to the
next paragraph. Otherwise, I recommend you to read the following post
[Implementing a Secure Token Service with
WCF](http://weblogs.asp.net/cibrax/archive/2006/03/14/440222.aspx)
first.

### 

### Informal definition of Federation

![](/images/legacy/SimpleFederation.jpg)

The image above illustrates a very simple view of a federation scenario.
In the first block, we have a realm or security domain. A realm is a
trust boundary that that can expand across several networks and it is
basically composed by client applications, an STS and a collection of
services (they essentially provide the business value to the client
applications). A group of connected realms  conform a federation
scenario (The trust boundary is around the STSs, each STS trust each
other).

You can read this article for a more formal definition of Federation,
[http://msdn2.microsoft.com/en-us/library/bb498017.aspx](http://msdn2.microsoft.com/en-us/library/bb498017.aspx)

### Key Interchange Process

**1. The client sends the RST message to the STS**

As part of the initial negotiation of the token with the STS through the
RST/RRST (Request Security Token / Response Request Security Token)
messages, the client application also negotiates with the STS a
cryptographic key that will be used later to secure the communication
with the final service.

This key can be symmetric or asymmetric, and the client application will
decide which kind of type it wants in the initial RST message.

This can be done in WCF by configuration:

\<wsFederationHttpBinding\>

  \<binding name="ServiceBinding"\>

  \<security mode="Message"\>

    \<message
issuedTokenType="http://docs.oasis-open.org/wss/oasis-wss-saml-token-profile-1.1\#SAMLV1.1"
**issuedKeyType="SymmetricKey"**\>

If the STS only supports one type of key (Symmetric or Asymmetric), that
is another story, the client should know that beforehand. The STS can
assume a default key type and always return that type or in the worst
case, return an exception message.

After generating the right key, the STS wraps it in an encrypted key
token
([http://www.ws-i.org/Profiles/BasicSecurityProfile-1.1.html\#EncryptedKeyToken](http://www.ws-i.org/Profiles/BasicSecurityProfile-1.1.html#EncryptedKeyToken))
using the public X509 key of the destination service (This is determined
by the AppliesTo element in the RST message), and adds the encrypted
token in the issued SAML token (This token is signed by the private X509
private key of the STS).

The plain key is also added in the RRST message so the client
application can get it from there (This assumes that the communication
with the STS is also secure).

**2. The client receives the RRST message and creates the request
message for the final service**

The client application receives the RRST message, and afterwards, it
extracts the SAML token and the generated key. Finally, it adds the SAML
token to the request message for the final service and secures the
channel (Encrypts the request) using the plain key received from the
STS.

**3. The service receives the request message from the client**

The service on the other side, receives the encrypted message and the
SAML token. After that, it unwraps the encrypted key token included in
the SAML token with its X509 private key. This key is used later to
decrypts the request message sent by the client application.

### Share a SAML token between several services

Ok, everything looks ok so far, the problems start here :).  If you go
back to the step 1, the STS will wrap the key into the SAML token using
a public key of a X509 certificate. Does this means that a SAML token
will only be useful for the service that possess the corresponding
private key ?. In other words, is the SAML token only valid for one
service ?. Ok, the answer will depend on a specific factor, is the
private X509 key shared among all the services ?. If the answer is YES,
the SAML token will be reusable for any of those services.

Now, is that a correct approach for implementing Single Sign-On ?. From
my point of view, it is not, it just a security hole in the system. If
one SAML token gets compromised by an attacker, this person will also
get access to any service that share the same key. On other hand, if we
have one key per service, if the SAML token gets compromised, only the
service associated with that token is compromised. Clear enough, isn't
it ? Single Sign-On here does not mean "Get a token once, use it
everywhere ...". It means, "use always the same set of credentials
against the STS to get the correponding token".

### Single Sign-On in Federation

In this section I will try to describe with my words what "Single
Sign-On" means in a federation scenario. If we have a client application
that wants to consume a service in another security realm, do we need to
have another set of credentials to log into that realm and consume the
service ?. The answer is "NO", if a trust relationship exists between
the Realms where the client application and the services are located, an
automatic negotiation of credentials can happen on behalf of the client
application between the STSs, and therefore the Single-Sign On is
archived.

If the STS in the second realm trusts the first realm, it can accepts
SAML tokens from the first STS (Since the SAML token is signed, it can
verify the origin with a X509 public key) and transform them to tokens
valid for the second realm. This is how federation works behind stage,
the trust relationship is delegated to the STSs.

Of course, I do not have the last word and any feedback is always
welcome :).

