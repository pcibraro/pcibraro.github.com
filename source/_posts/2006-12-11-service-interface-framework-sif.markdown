---
layout: post
title: "Service Invocation Framework"
date: 2006-12-11
comments: true
categories: .NET
---

My friends Sergio Borromei (Microsoft Consulting) and Rodolfo
Finochietti (Lagash) have recently released the first RTM of their
Service Invocation Framework.

This light-weight basically provides a collection of service adapters
that a client application can use to consume a service with some
location and implementation transparency.

The client application only need to know the service contract to locate
and consume the real service implementation, which can actually be a
remoting object, a web service or an in-proc object to name a few.

In my opinion, this kind of framework can be useful in scenarios
where you need to interop with different technologies. This helps to
minimize the dependency between the client application and those
technologies and facilitate possible changes due to new requirements. 

These are some of the adapters that comes by default with the framework:

-    Web Service Adapter: It can be used to execute any HTTP web service
    compliant with the Basic Profile. This adapter also provides a nice
    feature to generate the web service proxies on the fly using a
    wsdl definition. (Yes, there is not need to manually generate
    the proxies using a tool like wsdl.exe).
-   Remoting Adapter: As it name says, it can be used to execute methods
    on a object published with remoting.
-   InProc Adapter: This adapter is useful in scenarios where the client
    and service live in the same process, so any inter-process
    communication or data marshaling is not required.
-   Pipeline Adapter: It can execute a collection of filters to modify
    the service's inbound or outbound messages.

Let's go through a simple quickstart to show the main features of this
framework:

​1. First of all, we need to define a interface for our service
implementation. In this sample, I decided to design a extremely complex
one to return a "Hello World" message:

public interface IHelloWorld

{

    string HelloWorld();

}

​2. As a second step, our real service implementation needs to implement
the interface.  I used a web service, but it can be any other service
implementation.

[WebService(Namespace = "http://tempuri.org/")]

[WebServiceBinding(ConformsTo = WsiProfiles.BasicProfile1\_1)]

public class Service : System.Web.Services.WebService, IHelloWorld

{

    public Service () {

 

    }

 

    [WebMethod]

    public string HelloWorld() {

        return "Hello World";

    }

 

}

 

​3. Now, it is time to implement the client application. We have to add
a the following configuration section to the application configuration
file,

 

\<configuration\>

  \<configSections\>

    \<section name="servicesInvocationFramework"
type="InvocationFramework.Core.Configuration.SectionHandler,
InvocationFramework.Core, Version=1.1.0.0, Culture=neutral,
PublicKeyToken=4396b271bcbb0166" /\>

  \</configSections\>

 

  \<servicesInvocationFramework\>

    \<caller\>

      \<serviceAdapters\>

        \<serviceAdapter id="WebServicesAdapter" order="0"
enabled="true"
type="InvocationFramework.Adapters.WebServices.WebServicesAdapter,
InvocationFramework.Adapters.WebServices, Version=1.1.0.0,
Culture=neutral, PublicKeyToken=4396b271bcbb0166"\>

          \<services\>

            \<service name="SampleService"
wsdlLocation="http://localhost:9300/SampleService/Service.asmx?WSDL"
contractType="Contracts.IHelloWorld, Contracts"/\>

          \</services\>

        \</serviceAdapter\>

      \</serviceAdapters\>

    \</caller\>

  \</servicesInvocationFramework\>

\</configuration\>

In a few words, this section says that we are going to use the service
adapter to consume the service "SampleService". This service also
implements the contract IHelloWorld.

The WSDL location is required because the adapter will generate the
proxy to consume the service using that definition.

​4. Finally, some code is required on the client application to locate
the service and consume it.

ServiceProxyBuilder\<IHelloWorld\> factory = new
ServiceProxyBuilder\<IHelloWorld\>("SampleService");

IHelloWorld service = factory.BuildProxy();

 

Console.WriteLine(service.HelloWorld());

It is as simple as that, the client receives a proxy class to consume
the real service implementation. (All the inter-process communications
and messages interchanges will be managed by the adapter).

You can find more information about this framework in the [Rodolfo's
blog](http://weblogs.shockbyte.com.ar/rodolfof/archive/2006/12/08/10650.aspx),
and the [Service Invocation Framework web site
(CodePlex)](http://www.codeplex.com/SIF)

