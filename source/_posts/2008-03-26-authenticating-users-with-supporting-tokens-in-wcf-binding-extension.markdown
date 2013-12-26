---
layout: post
title: "Authenticating users with Supporting Tokens in WCF - Binding Extension"
date: 2008-03-26
comments: true
categories: WCF
---

A couple of months ago I described a useful authentication pattern for
Web applications based on supporting tokens, one of features provided by
WCF. After that, Dominick Baier wrote  a nice and intuitive
[article](http://www.leastprivilege.com/UserNameSupportingTokenInWCF.aspx)
showing this pattern in practice with real code examples, something I
could not include in my last post for time reasons :(.

A bad thing about supporting tokens is that they can only be configured
through code, there is no configuration support for this, which it means
that the security binding element has to be modified at some point to
include the supporting token requirements on the client and service side
(e.g, token type, encryption and signature requirements). If IIS is
being used as service host, this will eventually require a custom
service factory just to plug in those requirements. 

Fortunately, WCF supports configuration extension points to allow
developers to extend specific points of the configuration schema and
object model.

The \<extensions\> section provides a centralized location for the
registration of custom configuration objects required for creating a
relatively open extensibility point to the WCF configuration system. 
Each configuration extensibility point connects to one of three
collections within this section:  behaviorExtensions,
bindingElementExtensions, and bindingExtensions.

A really good thing about the configuration extensions is that the
developer does not have use any custom code like custom proxies or
service factories to load those security settings (All that work is done
by the extensions).

During this post I will show how to write a custom binding extension to
specify supporting token requirements for an endpoint through
configuration. This sample will be specific to the web application
scenario described by the pattern.

Let's first discuss the configuration schema we want to have for the
client and the service side. What we initially need is a custom binding
to support the mutual X509 authentication scenario,

\<customBinding\>

  \<binding name="MutualCertificateBinding"\>

    \<security authenticationMode="MutualCertificate"/\>

    \<httpTransport/\>

  \</binding\>

\</customBinding\>

Simple enough, this binding should be configured on the client and
service side. Secondly, we have to specify the X509 certificates that
will be used as client and service credentials for this binding.

On the client side, those credentials are specified by means of a
endpoint behavior,

\<behavior name="ClientBehavior"\>

  \<clientCredentials\>

    \<serviceCertificate\>

       \<defaultCertificate findValue="CN=SampleService"
storeLocation="LocalMachine" storeName="My"
x509FindType="FindBySubjectDistinguishedName"/\>

       \<authentication revocationMode="NoCheck"
certificateValidationMode="None"\>\</authentication\>

    \</serviceCertificate\>   \</clientCredentials\>

\</behavior\>

The equivalent configuration settings will be set  on the server side,
the same binding and a service behavior to specify the service
certificate.

\<behavior name="ServiceBehavior"\>

  \<serviceCredentials\>

    \<serviceCertificate findValue="CN=SampleService"
storeLocation="LocalMachine" storeName="My"
x509FindType="FindBySubjectDistinguishedName"/\>

    \<clientCertificate\>

      \<authentication revocationMode="NoCheck"/\>

    \</clientCertificate\>

  \</serviceCredentials\> \</behavior\>

Once we have these standard configuration, the next step is to add the
custom extension (On both sides, the client and the service) and
reference the existing "MutualCertificate" binding from there (The one
that we previously defined). The custom extension will basically add the
supporting token requirements to that binding.

\<extensions\>     \<bindingExtensions\>           \<add
name="trustedWeb" type="Samples.TrustedWeb, Samples"/\>    
\</bindingExtensions\>

\</extensions\>

\<bindings\>

    \<trustedWeb\>

     \<binding name="MyTrustedWeb" 
bindingReference="MutualCertificateBinding" /\>

    \</trustedWeb\>

    \<customBinding\>

      \<binding name="MutualCertificateBinding"\>

        \<security authenticationMode="MutualCertificate"/\>

        \<httpTransport/\>

      \</binding\>

    \</customBinding\>

\</bindings\>

The client and service will have to use this binding instead of the
original mutual certificate binding (The one configured in the
originalBinding attribute).

Client configuration:

\<client\>

  \<endpoint address="http://localhost/SampleService/Service.svc"

    binding="trustedWeb" bindingConfiguration="MyTrustedWeb"
behaviorConfiguration="ClientBehavior"

   contract="Client.ISampleService"\>

  \</endpoint\>

\</client\>

Service configuration:

 \<services\>

   \<service name="SampleService"
behaviorConfiguration="ServiceBehavior"\>

     \<endpoint binding="trustedWeb" address=""
bindingConfiguration="MyTrustedWeb" contract="ISampleService"\>

\</endpoint\>

\</service\>

\</services\>

The trustedWeb custom extension performs two things:

​1. Load the binding referenced by the attribute "bindingReference"

protected override void OnApplyConfiguration(Binding binding)

{

  TrustedWebClientBinding trustedWebClient =
(TrustedWebClientBinding)binding;

  BindingsSection bindings =
(BindingsSection)ConfigurationManager.GetSection("system.serviceModel/bindings");

  if (bindings == null)

  {

    throw new ConfigurationErrorsException("Unexisting bindings
section");

  }

 
if(!bindings.CustomBinding.Bindings.ContainsKey(this.BindingReference))

    throw new ConfigurationErrorsException(string.Format("Unexisting
binding configuration {0}", this.BindingReference));

  CustomBindingElement element =
bindings.CustomBinding.Bindings[this.BindingReference];

  trustedWebClient.Binding = new CustomBinding();

  element.ApplyConfiguration(trustedWebClient.Binding);

}

​2. Creates the binding elements used by the referenced binding and adds
the Supporting tokens requirements for an username.

public override BindingElementCollection CreateBindingElements()

{

    return AddUserNameSupportingTokenToBinding(Binding);

}

private BindingElementCollection
AddUserNameSupportingTokenToBinding(Binding binding)

{

  BindingElementCollection elements = binding.CreateBindingElements();

  SecurityBindingElement security =
elements.Find\<SecurityBindingElement\>();

  if (security != null)

  {

    UserNameSecurityTokenParameters tokenParameters = new
UserNameSecurityTokenParameters();

    tokenParameters.InclusionMode =
SecurityTokenInclusionMode.AlwaysToRecipient;

    tokenParameters.RequireDerivedKeys = false;

   
security.EndpointSupportingTokenParameters.SignedEncrypted.Add(tokenParameters);

    return elements;

  }

  throw new ArgumentException("Unexisting security binding element");

}

The complete code is available to download from
[here](/images/legacy/TrustedWeb.zip).

