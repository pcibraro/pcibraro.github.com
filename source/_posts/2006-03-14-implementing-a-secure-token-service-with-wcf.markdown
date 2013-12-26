---
layout: post
title: "Implementing a Secure token service with WCF"
date: 2006-03-14
comments: true
categories: WCF
---

I decided to write this post in order to show some necessary steps to
build a Secure Token Service (STS) with the latest WCF CTP.

There is a lot of messages in the newsgroups from people with problems
to implement a solution like this, so they may find this article useful.

The image below illustrates a generic architecture for an application
that uses a brokered authentication with a STS.

 

 

 

 ![](/images/legacy/ArchitectureWCFSTS.gif)

 

The client application is using a customBinding to secure the
communication with the STS and a wsFederationHttpBinding to do that with
the target service. In this case, the wsFederationHttpBinding includes
the token obtained from the STS in the request message for the service.

As you can see in the image, the following steps are performed in order
to execute the final service:

 

​1. The client application sends a RequestSecurityTokenMessage (RST) to
the STS according to WS-Trust specification.

​2. The STS receives a RST message, extract some information from it and
creates a token. After that, it sends back a
RequestSecurityTokenResponseMessage (RSTR) with the new token.

​3. The client application sends a request message to the service and
includes the token obtained from the STS.

​4. The service executes the service and returns the response to the
client application. The token is used to build the security claims for
the authenticated user before calling the service method.

 

 

**WCF configuration for the client application**

 

 

\<system.serviceModel \>

        \<client\>

            \<!-- Endpoint configuration --\>

            \<endpoint name="clientendpoint"
address="http://localhost/WCFSampleService/service.svc"

                binding="wsFederationHttpBinding"

                contract="IHelloWorld"

                behaviorConfiguration="ServiceBehavior"

                bindingConfiguration="ServiceBinding"\>

                \<identity\>

                    \<dns value="WCFQuickstartServer"/\>

                \</identity\>

            \</endpoint\>

        \</client\>

 

        \<bindings\>

 

            \<!-- Binding used to secure the communication with the STS
--\>

            \<customBinding\>

                \<binding name="UsernameBinding"\>

                    \<security
authenticationMode="UserNameForCertificate"

                            requireSecurityContextCancellation ="false"

                            requireSignatureConfirmation="false"

                            messageProtectionOrder
="SignBeforeEncryptAndEncryptSignature"

                            requireDerivedKeys="true"\>

                    \</security\>

                    \<httpTransport/\>

                \</binding\>

            \</customBinding\>

 

            \<!-- Binding used to secure the communication with the
service --\>

            \<wsFederationHttpBinding\>

                \<binding name="ServiceBinding"\>

                    \<security mode="Message"\>

                        \<message
issuedTokenType="http://docs.oasis-open.org/wss/oasis-wss-saml-token-profile-1.1\#SAMLV1.1"
negotiateServiceCredential="false"\>

                            \<!-- Uncomment this section to ask for
specific claims to the STS

                            \<claims\>

                                \<add claimType 
="http://schemas.microsoft.com/ws/2005/05/identity/claims/EmailAddress"/\>

                                \<add claimType 
="http://schemas.microsoft.com/ws/2005/05/identity/claims/GivenName"/\>

                                \<add claimType 
="http://schemas.microsoft.com/ws/2005/05/identity/claims/Surname"/\>

                            \</claims\>

                            --\>

 

                            \<!-- Information related to the Secure
token service --\>

                            \<issuer
address="http://localhost/WCFSecurityTokenService/service.svc"
bindingConfiguration="UsernameBinding"

                                binding="customBinding"\>

                                \<identity\>

                                    \<dns value="WCFQuickstartServer"/\>

                                \</identity\>

                            \</issuer\>

                        \</message\>

                    \</security\>

                \</binding\>

            \</wsFederationHttpBinding\>

        \</bindings\>

        \<behaviors\>

            \<!-- Credentials configuration --\>

            \<behavior name="ServiceBehavior"\>

                \<clientCredentials\>

                    \<serviceCertificate\>

                        \<defaultCertificate
findValue="CN=WCFQuickstartServer" storeLocation="LocalMachine"
storeName="My" x509FindType="FindBySubjectDistinguishedName"/\>

                        \<authentication revocationMode="NoCheck"
certificateValidationMode="None"\>\</authentication\>

                    \</serviceCertificate\>

                \</clientCredentials\>

            \</behavior\>

        \</behaviors\>

 \</system.serviceModel\>

 

 

****

Some notes about the configuration above:

 

​1. The communication between the client and the STS is secured by a
UsernameForCertificate binding. That is, the STS expects a UsernameToken
as client token (Token used to authenticate the client)  and a
X509Certificate as service token (Token used to encrypt and sign the
message).

​2. The "issueTokenType" attribute in the "message" element specifies
the token type expected by the Service. The client application will
include that value in the RST message and therefore the STS will know
what kind of token it must create. If the STS does not support that kind
of token, then it will return a fault message. For this sample, the
client application is asking for a SAML token.

​3. The "negotiateServiceCredential" attribute in the "message"
element specifies if the client must interchange additional messages
with the STS in order to negotiate the service certificate. I will give
more information about this flag later in the next paragraphs.

​4. The "address" attribute in the "issuer" element specifies the
address of the STS. WCF also includes a default implementation of a
InfoCard STS. In order to use that STS, you must configure the address
[http://schemas.microsoft.com/ws/2005/05/identity/issuer/self](http://schemas.microsoft.com/ws/2005/05/identity/issuer/self).

​5. The claims element is only valid for SAML tokens. It specifies what
claims are expected in the token.

 

 

**WCF configuration for the STS**

 

 

\<system.serviceModel\>

        \<services\>

            \<service behaviorConfiguration="ServiceBehavior"
name="MySecureTokenService"\>

                \<endpoint binding="customBinding" address=""
bindingConfiguration="ServiceBinding"
contract="IMySecureTokenService"\>\</endpoint\>

            \</service\>

        \</services\>

        \<bindings\>

            \<customBinding\>

                \<binding name="ServiceBinding"\>

                    \<security
authenticationMode="UserNameForCertificate"

                            requireSecurityContextCancellation ="false"

                            requireSignatureConfirmation="false"

                            messageProtectionOrder
="SignBeforeEncryptAndEncryptSignature"

                            requireDerivedKeys="true"\>

                    \</security\>

                    \<httpTransport/\>

                \</binding\>

            \</customBinding\>

        \</bindings\>

        \<behaviors\>

            \<behavior name="ServiceBehavior"
returnUnknownExceptionsAsFaults="false"\>

                \<serviceCredentials\>

                    \<serviceCertificate
findValue="CN=WCFQuickstartServer" storeLocation="LocalMachine"
storeName="My" x509FindType="FindBySubjectDistinguishedName"/\>

                    \</serviceCredentials\>

            \</behavior\>

        \</behaviors\>

 \</system.serviceModel\>

 

 

In this case, the STS implementation is in the class
"MySecureTokenService" and it exposes the contract
"IMySecureTokenService".

The binding configuration is similar to the configuration in the client.

 

 

**WCF configuration for the Service**

 

 

****

\<system.serviceModel\>

        \<services\>

            \<service

                behaviorConfiguration="ServiceBehavior"

                name="SampleService.HelloWorldService"\>

                \<endpoint binding="wsFederationHttpBinding"

                    address=""

                    bindingConfiguration="ServiceBinding"

                    contract="SampleService.IHelloWorld"/\>

            \</service\>

        \</services\>

        \<bindings\>

            \<wsFederationHttpBinding\>

                \<binding name="ServiceBinding"\>

                    \<security mode="Message"\>

                        \<message
issuedTokenType="http://docs.oasis-open.org/wss/oasis-wss-saml-token-profile-1.1\#SAMLV1.1"
negotiateServiceCredential="false"\>

                            \<!--\<claims\>

                                \<add
claimType="http://schemas.microsoft.com/ws/2005/05/identity/claims/EmailAddress"/\>

                                \<add
claimType="http://schemas.microsoft.com/ws/2005/05/identity/claims/GivenName"/\>

                                \<add
claimType="http://schemas.microsoft.com/ws/2005/05/identity/claims/Surname"/\>

                            \</claims\>--\>

                            \<issuer
address="http://localhost/SamlSecurityTokenService/SamlTokenIssuer.ashx"
bindingConfiguration="UsernameBinding" binding="customBinding"\>

                                \<identity\>

                                    \<dns value="WCFQuickstartServer"/\>

                                \</identity\>

                            \</issuer\>

                        \</message\>

                    \</security\>

                \</binding\>

            \</wsFederationHttpBinding\>

        \</bindings\>

        \<behaviors\>

            \<behavior name="ServiceBehavior"
returnUnknownExceptionsAsFaults="false"\>

                \<serviceCredentials\>

                    \<serviceCertificate
findValue="CN=WCFQuickstartServer" storeLocation="LocalMachine"
storeName="My" x509FindType="FindBySubjectDistinguishedName"/\>

                \</serviceCredentials\>

 

            \</behavior\>

        \</behaviors\>

 \</system.serviceModel\>

 

 

Again, the configuration of the wsFederationHttpBinding is identical to
the configuration in the client application.

 

 

**STS implementation**

 

 

****

The contract for the STS is quite simple and looks as follows:

 

 

[ServiceContract]

public interface IMySecurityTokenService

{

    [OperationContract(Action =
"[http://schemas.xmlsoap.org/ws/2005/02/trust/RST/Issue](http://schemas.xmlsoap.org/ws/2005/02/trust/RST/Issue)",

                      ReplyAction =
"http://schemas.xmlsoap.org/ws/2005/02/trust/RSTR/Issue")]

        Message IssueToken(Message rstMessage);

}

 

 

It exposes one method "IssueToken" for the action
"http://schemas.xmlsoap.org/ws/2005/02/trust/RST/Issue" that is part of
the WS-Trust specification.

 

 

public class MySecureTokenService : IMySecurityTokenService

{

        public MySecureTokenService()

        {

        }

 

        public Message IssueToken(Message rstMessage)

        {

            RequestSecurityToken rst =
RequestSecurityToken.CreateFrom(rstMessage.GetReaderAtBodyContents());

 

            SecurityToken issuedToken = null;

 

            //Code to create the token goes here ......

 

            // setup RSTR

            RequestSecurityTokenResponse rstr = new
RequestSecurityTokenResponse();

 

            //attach security token to RSTR

            rstr.RequestedSecurityToken = issuedToken;

            rstr.TokenType = rst.TokenType;

 

            // send RSTR

            rstr.MakeReadOnly();

            Message rstrMessage =
Message.CreateMessage(rstMessage.Version,
"http://schemas.xmlsoap.org/ws/2005/02/trust/RSTR/Issue", rstr);

            rstrMessage.Headers.RelatesTo =
rstMessage.Headers.MessageId;

 

            return rstrMessage;

        }

    }

 

 

The STS implementation receives a generic message containing the RST and
creates a token using that information.\
At the end, it returns a message containing the RSTR with the issued
token. For this sample, I have omitted the code to build the token since
you can create any token there (UsernameToken, SamlSecurityToken,
etc)  depending on the value of the property "rst.TokenType".

 

 

**Avoiding the service credential negotiation**

 

****

WCF provides a new feature to negotiate the service credentials for a
service.

When this feature is turned on, the client does not need to manually
configure or specify the service credentials for the service. As a
result, the client application interchanges an additional message with
the service using a a protocol called SP-Nego.

There is not any documentation or information around for that protocol,
so it could be a problem if want to host your service in different
platform like WSE. That is not a problem in WCF because the security
bindings know how to interpret this message and create a response
according to its content.

 

There are two ways to disable this feature in WCF:

 

1. Secure the communication with a customBinding since it does
not provide this feature.

​2. Turn off the attribute "negotiateServiceCredential" in the "message"
element for the bindings wsFederationHttpBinding or wsHttpBinding.

 

UPDATE: The STS implementation code is available in this post
[http://weblogs.asp.net/cibrax/archive/2006/09/08/SAML-\_2D00\_-STS-implementation-for-WCF.aspx](http://weblogs.asp.net/cibrax/archive/2006/09/08/SAML-_2D00_-STS-implementation-for-WCF.aspx)



