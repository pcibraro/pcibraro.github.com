---
layout: post
title: "Services in .NET - Part I"
date: 2007-01-25
comments: true
categories: WCF
---

**What is a service after all ?**

I could not find a concrete definition of service. Many authors give
different definitions for the same thing but all of them agree on
describing a service through the famous tenets of service orientation:

​1. Services are autonomous

​2. Service have explicit boundaries

​3. Services share schema and contract, but do not share implementation

​4. Service compatibility is determined by a policy

After reading these tenets I can conclude the following:

-   Each service is independent of other services, for instance, if one
    service crashes, the other services are not affected.
-   At first glance, I know where a service ends and another starts.
-   A service can be considered as an unit of design and deployment.
-   Schema describes the format and the content of the messages.
    Contract describes message sequences allowed in and out of the
    service. This is more related to the concept of loose-coupling
    through messaging (The client and the service are decoupled), the
    communication between services is performed by means of a message
    interchange pattern. For instance, this is not true for a .NET
    remoting object where the client must have a reference to the server
    type.
-   The use of a service is governed by metadata or policies. In other
    words, each service has a set of requirements to be used, and these
    requirements are expresses through metadata. For instance security.

This is nice in theory, but not-so-clever in practice.

As you can see, these tenets do not mention anything about XML, SOAP or
any of the available WS-\* standards. Therefore, I assume that a Soap
Service is a concrete implementation that adhere to these principles.
Let's discuss more about soap service in the following paragraphs.

**Soap Services**

Soap Services interchange xml messages (Soap Messages) which basically
contains data (The payload expected by the service) and metadata
(Headers required to execute the service, most of them are part the
WS-\* Family). The service policy in this kind of service is expressed
through the WS-Policy standard, and the contract through WSDL.

The concept of service becomes evident in WCF, where we can build real
soap services. According to the Don Box's article "[A Guide to
Developing and Running Connected Systems with
Indigo](http://msdn.microsoft.com/msdnmag/issues/04/01/indigo/default.aspx#S3)",  

*Service-oriented development focuses on systems that are built from a
set of autonomous services. This difference has a profound impact on the
assumptions one makes about the development experience.*

*In Indigo, a service is simply a program that one interacts with via
message exchanges. A set of deployed services is a system.*

In addition, a web service is an specific type of Soap Service that uses
Http as transport for interchanging messages. The Web services (Asmx)
provided by default in the .NET framework (Basic Profile 1.1)  does not
implement the concept of policy at all, unless you decided to use WSE or
WCF.

**Existing techniques to design and implement a soap service**

Once we know the business requirements for a service, it is time to
start thinking in the design and implementation. Nowadays, the most
common paths in that direction are "Development-First" and
"Contract-First".

In the first approach or technique, which is the approach used by
default when you develop an asmx service with Visual Studio, the steps
to design the XSD and WSDL are totally hidden for the developer.

The developer begin authoring the service class and its operations or
methods (A asmx for simple web services or a service contract in WCF).
In addition, he decorates the service with attributes that the framework
will use later to automatically generate the WSDL for the service. In
this way, the developer does not have to deal with complexities of WSDL
or XSD and he can get the service running with some minimal efforts.  

On the other hand, the "Contract-First" approach has a completely
different starting point. As first step in this approach, the developer
usually defines the XSD types and designs the service messages.  

Once the service messages are ready, as second step, he begin authoring
the service WSDL or contract. Finally, the latest service's WSDL version
is used to generate the code and implementation.  This can be easily
done in .NET with the WSDL.exe tool, but other platforms such as JAVA
also provides tools to perform the same task. Most developers use this
approach from a client side perspective to generate the proxy classes
required to consume the service.

As you can imagine, creating a WSDL by hand can be a complicated and
error prone task, many developers (including me) are not familiar at all
with the complete specification. However, it provides a explicit control
of the generated contracts, which is in my opinion, it is a crucial
point for interop scenarios.

Unfortunately, these is not a good tooling support in Visual Studio to
use this approach, I mean VS does not provide the necessary tools
to increase the developer productivity.

As result, the .NET community has started developing tools to support
his approach as well. One of the most well-know projects in this are is
[WSCF](http://www.thinktecture.com/Resources/Software/WSContractFirst/default.html)
created by [Thinktecture](http://www.thinktecture.com), this tool
basically provides a set of wizards and designers to generate the XSD
types and the final WSDL.

The [Service Factory Project](http://msdn.microsoft.com/servicefactory)
from Patterns & Practices supports both approaches as well, if you are
not familiar with project, the P&P team has lately beginning to provide
software factories for different purposes, most of them are based on the
Guidance Automation Toolkit (GAT). The aim of this project is to create
web services using the best practices of the market and facilitate the
developer work through a set of tools, wizards and code generators
totally integrated in Visual Studio. Don Smith has published a [nice
blogcast](http://blogs.msdn.com/donsmith/archive/2006/04/20/579735.aspx\)
showing the complete process to create contracts with the wizards
provided in Service Factory.

As usual, there is some controversy about what approach is the
best, [Dare Obasanjo](http://www.25hoursaday.com) gives his thoughts in
this [interesting
post](http://www.25hoursaday.com/weblog/PermaLink.aspx?guid=e1ab8978-f0a9-4913-bee3-badc1cbefbe5).

I personally prefer a mix between the two approaches, which can be
illustrated with the following steps:

1.  Design the XSD types and messages
2.  Create the service interface with the necessary operations
3.  Decorate the interface and operations with the right attributes
4.  Use the WSDL generation support provided by ASP.NET or WCF to
    automatically create the service WSDL

We (Me and rest of team) successfully used this approach in the ["WS-I
Sample Application"](http://practices.gotdotnet.com/wsibsp) project. The
main objective here was interop between different platforms, and I can
say it worked perfect, we only found some minor issues without
importance.

For more information about this topic, I recommend these two articles
written by Aaron Skonnard:

-   [Contract-First Service
    Development](http://msdn.microsoft.com/msdnmag/issues/05/05/ServiceStation/#S3)
-   [Techniques for Contract-First
    Development](http://msdn.microsoft.com/msdnmag/issues/05/06/ServiceStation/)

 **Service Versioning**

For this section, I decided to put a link to this excellent article
["Versioning Web
Services"](http://blogs.msdn.com/donsmith/pages/VersioningWebServices.aspx)
 written by Don Smith. This article mainly discusses well-know practices
to the versioning problem in Soap Services.

This is all for now, I will continue discussing more topics about
service design in my next posts.

