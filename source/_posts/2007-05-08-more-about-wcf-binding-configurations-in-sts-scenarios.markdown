---
layout: post
title: "More about WCF Binding configurations in STS scenarios"
date: 2007-05-08
comments: true
categories: WCF
---

Some time ago I wrote an [introductory
post](http://weblogs.asp.net/cibrax/archive/2006/03/14/440222.aspx)
showing a basic architecture for a federation scenario in WCF. If we go
back until then, the architecture showed there was something like the
image below:

![](/images/legacy/STS_WCF_Bindings.jpg)

In this simple architecture we can find the following components:

1.  Client Application: The application that will be consuming the WCF
    services. In order to execute one of those services, this
    application needs first a security token granted by the STS.
2.  Security Token Service (STS): It is an specific service with the
    unique purpose to grant tokens according to some information
    received from the client. The process to grant tokens is not as easy
    as it sounds, I will talk more about this later in this post.
3.  Service: An specific service that provides some functionality
    required by the client application, for instance, a business
    operation. There is an implicit trust relationship between the
    service and the STS, any valid token granted by the STS can be used
    to authenticate the client.

Now that we have a better understanding of the overall architecture, I
will discuss more in detail the WCF bindings involved in the process to
interchange the messages between the different parties.

### Communication between the client and the STS

At this point, a pair of RST/RSTR messages will be interchanged between
both parties. The WCF infrastructure takes care of creating those
messages for us, but we still need to configure some channel settings
through a binding configuration. Some of those settings involve channel
security, type of transport or more specific things like the type of
security token that will be presented to the STS. 

As you can see, there is not any limitation here and we are free to use
any of the existing WCF bindings.

In my previous sample I used a mutual certificate authentication over
Http. Anyway, I could also use any transport or any other kind of token
such as username token, a saml token, kerberos or any custom token.

```csharp
<customBinding>
  <binding name="MutalCertificateBinding">
    <security authenticationMode="MutualCertificate" 
         requireSecurityContextCancellation ="false"
         requireSignatureConfirmation="false"
         messageProtectionOrder ="SignBeforeEncrypt"
    requireDerivedKeys="true">
    </security>
    <httpTransport/>
   </binding>
</customBinding>
```

### Communication between the client and the service

Here, we do not have the same flexibility as we had before, and
WsHttpFederation is the only available binding.

As it name clearly says, this binding only supports the following
aspects:

-   Authentication based on the token obtained from the configured STS
-   Http transport, which means that we can not use other kind of
    transport such as TCP, MSMQ or whatever we want.
-   Channel Security based on the token obtained from the STS.   

### 

```csharp
<wsFederationHttpBinding>
  <binding name="ServiceBinding">
    <security mode="Message">
      <message issuedTokenType="http://docs.oasis-open.org/wss/oasis-wss-saml-token-profile-1.1#SAMLV1.1" negotiateServiceCredential="false">
      <issuer address="http://localhost/WCFSecurityTokenService/service.svc" bindingConfiguration="MutalCertificateBinding" binding="customBinding">
        <identity>
    <dns value="WCFQuickstartServer"/>
        </identity>
      </issuer>
     </message>
    </security>
  </binding>
</wsFederationHttpBinding>
```

### Other alternatives for the communication between the client and the STS

A typical case could be to present a token obtained from a second STS in
our STS. This scenario is usually called Federation.

The image below illustrates the architecture for this kind of scenario:

![](/images/legacy/Federation_WCF_Binding.jpg)

What is really important about federation is that a trust relationship
can be expanded across different domains without limits through
different STS.

The WCF SDK already provides a good sample of this scenario, so you can
find the code here [SDK Folder]\\TechnologySamples\\Scenario\\Federation

