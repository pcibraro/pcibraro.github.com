---
layout: post
title: "Federation Over TCP With WCF"
date: 2008-04-21
comments: true
categories: Federation
---

One of the discussions that we had during the last summit with the rest
of "Connected Systems" MVPs was the possibility of supporting a
Federation Scenario over TCP in WCF. For many of us that scenario was
possible in theory, but unfortunately no documentation or samples
existed to support it. In fact, WCF only comes with pre-built binding
for federation scenarios, the "WsFederationHttpBinding" binding, which
is completely tied to Http.

For that reason, I decided to give it a shot and try to manipulate some
custom bindings to use tcp instead of the common used http transport.
One curios thing about TCP is that it requires security sessions
(SecureConversation with requireSecurityContextCancellation equals to
"True") in order to work fine. If you do not configure the binding with
those security settings, WCF will throw a nice error message saying that
the order of the binding elements is not correct. At the beginning I did
not configure it in that way, and it took me sometime to figure out what
the problem was, I would save some time with a better error
description. 

The resulting bindings for client, STS and the sample service were the
following (In this sample, the client is authenticating against the
service with a client certificate).

​1. Client

\<bindings\>

  \<customBinding\>

    \<binding name="STSBinding"\>

      \<security authenticationMode="SecureConversation"
requireSecurityContextCancellation="true"\>

        \<secureConversationBootstrap
authenticationMode="MutualCertificate"/\>

      \</security\>

      \<binaryMessageEncoding/\>

      \<tcpTransport /\>

    \</binding\>

    \<binding name="ServiceBinding"\>

       \<security authenticationMode="SecureConversation"\>

         \<secureConversationBootstrap
authenticationMode="IssuedToken"\>

           \<issuedTokenParameters
tokenType=[http://docs.oasis-open.org/wss/oasis-wss-saml-token-profile-1.1\#SAMLV1.1](http://docs.oasis-open.org/wss/oasis-wss-saml-token-profile-1.1#SAMLV1.1)\>

             \<issuer address="net.tcp://localhost:8000/sts"
bindingConfiguration="STSBinding" binding="customBinding"\>

               \<identity\>

                 \<dns value="STSAuthority"/\> \<!--Sample Cert for the
STS --\>

               \</identity\>

             \</issuer\>

           \</issuedTokenParameters\>

        \</secureConversationBootstrap\>

      \</security\>

      \<binaryMessageEncoding/\>

      \<tcpTransport /\>

    \</binding\>

  \</customBinding\>

\</bindings\>

​2. STS

\<bindings\>

  \<customBinding\>

    \<binding name="MutualCertificateBinding"\>

      \<security authenticationMode="SecureConversation"
requireSecurityContextCancellation="true"\>

        \<secureConversationBootstrap
authenticationMode="MutualCertificate"/\>

      \</security\>

      \<binaryMessageEncoding/\>

      \<tcpTransport /\>

    \</binding\>    \</customBinding\>

\</bindings\>

​3. Sample Service

\<bindings\>

  \<customBinding\>

    \<binding name="SampleService"\>

      \<security authenticationMode="SecureConversation"
requireSecurityContextCancellation="true"\>

        \<secureConversationBootstrap authenticationMode="IssuedToken"\>

           \<issuedTokenParameters
tokenType=[http://docs.oasis-open.org/wss/oasis-wss-saml-token-profile-1.1\#SAMLV1.1](http://docs.oasis-open.org/wss/oasis-wss-saml-token-profile-1.1#SAMLV1.1)\>

        \</issuedTokenParameters\>

       \</secureConversationBootstrap\>

     \</security\>

     \<binaryMessageEncoding/\>

    \<tcpTransport /\>

   \</binding\>

  \</customBinding\>

\</bindings\>

It is not required that the STS and service use both TCP transport for
communicating with the client, which is a cool thing because now we can
combine different transports in a whole federation scenario. For
instance, we can have a Http communication between the client and the
STS, and a TCP communication with between the client and the final
service.

The complete sample is available to download from
[here](/images/legacy/FederationOverTcp.zip).

 

