---
layout: post
title: "Contract Projections in WCF declarative services"
date: 2009-02-18
comments: true
categories: WCF
---

As my friend Jesus mentioned in the post "[Using XAML serialization in
WCF
4.0](http://weblogs.asp.net/gsusx/archive/2008/12/17/using-xaml-serialization-in-wcf-4-0.aspx)",
WCF 4.0 introduces a new way to implement services that are totally
defined in XAML, which receive the name of "declarative services". In
the past, creating a simple service involved three basic steps,

1.  Define the service contract
2.  Implement the service contract
3.  Host the service implementation

\#1 and \#2 were all done in an imperative .NET programming language
such as C\# or VB.NET. Today, thanks to this new feature, we will able
to define the service interface (\#1) using XAML and implement the
service (\#2) using a declarative workflow (XAML too).

This was announced as part of the Microsoft's Oslo modeling vision in
the last PDC.

Aaron Skonnard has recently written an excellent article for the MSDN,
"WCF and WF Services in the .NET framework 4.0, and Dublin", where he
discusses all these new features more in detail, and the role of dublin
in that vision. Something I found interesting in that article was the
fact that he mentioned "Contract Projections" as part of "declarative
services".

A contract projection allows separating the logical contract definition
from the representation of the messages that are sent or received. We
will able to have a single contract definition and specify different
messaging styles like "SOAP" or "REST/POX" using a contract projection.

As in the example shown in the article, a regular WCF service definition
for a calculator service made in C\# would look like this,

~~~~ {#ctl00_mainContentContainer_ctl15 .libCScode style="WORD-BREAK: break-all; WORD-WRAP: break-word" space="preserve"}
[ServiceContract]
public interface ICalculator
{
    [OperationContract]
    int Add(int Op1, int Op2);
    [OperationContract]
    int Subtract(int Op1, int Op2);
};
~~~~

The equivalent representation in XAML (using declarative services) would
look like this,

~~~~ {#ctl00_mainContentContainer_ctl16 .libCScode style="WORD-BREAK: break-all; WORD-WRAP: break-word" space="preserve"}
<ServiceContract Name="ICalculator">
    <OperationContract Name="Add">
        <OperationArgument Name="Op1" Type="p:Int32" />
        <OperationArgument Name="Op2" Type="p:Int32" />
        <OperationArgument Direction="Out" Name="res1" Type="p:Int32" />
   </OperationContract>
   <OperationContract Name="Subtract">
        <OperationArgument Name="Op3" Type="p:Int32" />
        <OperationArgument Name="Op4" Type="p:Int32" />
        <OperationArgument Direction="Out" Name="res2" Type="p:Int32" />
   </OperationContract>
</ServiceContract>
~~~~

And finally, the projection of that contract at wire level like SOAP,

~~~~ {#ctl00_mainContentContainer_ctl17 .libCScode style="WORD-BREAK: break-all; WORD-WRAP: break-word" space="preserve"}
<Service.KnownProjections>
    <SoapContractProjection Name="ICalculatorSoapProjection">
        <!-- service contract definition goes here -->
    </SoapContractProjection>
</Service.KnownProjections>
~~~~

As you can see, we will able to have a single service implementation (or
XAML workflow), and multiple contract projections or "KnownProjections"
(for the different messaging styles) to get access to that service.

With this new feature, it looks like REST/POX will be officially
supported for consuming declarative services. (I talked in [the
past](http://weblogs.asp.net/cibrax/archive/2008/10/24/rest-and-workflow-services-play-well-together.aspx)
about exposing workflow services with REST).

As part of the PDC bits (The code that was distributed in a VPC), there
is a interface "IContractProjection" for defining new kind of
projections,

public interface IContractProjection \
{ \
    // Methods \
    void ApplyEndpointBehavior(ServiceEndpoint endpoint); \
    ContractDescription GetContractDescription();

    // Properties \
    string ConfigurationName { get; } \
    ServiceContract Contract { get; set; } \
}

For the moment, there is single implementation for Soap,
"SoapContractProjection". I do not know, we will see if the WCF/WF team
provide more implementations in the future.

