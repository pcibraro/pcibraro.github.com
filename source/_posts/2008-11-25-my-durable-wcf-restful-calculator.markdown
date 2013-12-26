---
layout: post
title: "My durable WCF RESTful calculator"
date: 2008-11-25
comments: true
categories: REST
---

A durable service in WCF is by a definition a service that can persist
all its internal state across calls in some durable storage. For every
operation, the service state is retrieved from the storage, the
operation is executed and finally the state is persisted again in the
storage. Therefore, there is not need to keep the service instance idle
in memory while waiting for client calls. It is equivalent to a long run
session, which make this feature something ideal for long-running
processes like workflows (In fact, workflow services are mount on top of
this feature),

In order to create a durable service, WCF provides a "DurableService"
attribute (It's a service behavior) that can be applied to a regular
service definition. The service itself has to be either serializable or
have members decorated with DataContract or DataMember attributes to be
serialized and stored in the persistent storage.

The service activation, as in workflow services, is managed by the WCF
context correlation mechanism. Once a service instance has been created,
the client application has to propagate some context information(which
includes the service instance id) in order to route all the new messages
to the right service instance. Jesus has already discussed how this
mechanism works more in detail in this
[post](http://weblogs.asp.net/gsusx/archive/2007/06/14/orcas-durable-services.aspx)
(Although the post is bit old and some names have changed since then, it
is worth reading).

For purposes of this post, I decided to create a simple calculator
example that exposes different operations through the classic http
verbs,

[ServiceContract(Namespace =
"http://Microsoft.WorkflowServices.Samples")]

public interface ICalculator

{

    [OperationContract()]

    [WebInvoke(Method = "POST")]

    int PowerOn();

 

    [OperationContract()]

    [WebInvoke(Method = "PUT", UriTemplate = "add")]

    int Add(int value);

 

    [OperationContract()]

    [WebInvoke(Method = "PUT", UriTemplate = "subtract")]

    int Subtract(int value);

 

    [OperationContract()]

    [WebInvoke(Method = "PUT", UriTemplate = "multiply")]

    int Multiply(int value);

 

    [OperationContract()]

    [WebInvoke(Method = "PUT", UriTemplate = "divide")]

    int Divide(int value);

 

    [OperationContract()]

    [WebInvoke(Method = "DELETE")]

    void PowerOff();

} 

The implementation of this service is also quite straightforward.

[Serializable]

[DurableService]

public class DurableCalculator : ICalculator

{

    int \_currentValue = 0;

 

    [DurableOperation(CanCreateInstance=true)]

    public int PowerOn()

    {

        return \_currentValue;

    }

 

    [DurableOperation]

    public int Add(int value)

    {

        return (\_currentValue += value);

    }

 

    [DurableOperation]

    public int Subtract(int value)

    {

        return (\_currentValue -= value);

    }

 

    [DurableOperation]

    public int Multiply(int value)

    {

        return (\_currentValue \*= value);

    }

 

    [DurableOperation]

    public int Divide(int value)

    {

        return (\_currentValue /= value);

    }

 

    [DurableOperation(CompletesInstance=true)]

    public void PowerOff()

    {

    }

}

As you can see, I decorated the service implementation with the
"DurableService" and "DurableOperation" attributes to make this simple
service a durable one.

WCF only comes with built-in support for transferring the context
information between the client and service with Http Cookies or Soap
Headers. While cookies would be the right mechanism for http REST
services, unfortunately they do not work as expected. The path that WCF
uses for creating the cookies is relative, so the context manager throws
the following exception on the client side,

***Unhandled Exception: System.Net.CookieException: An error occurred
when parsing the Cookie header for Uri
'http://localhost:8080/DurableCalculator'. ---\>
System.Net.CookieException: The 'Path'='/DurableCalculator/PowerOn' part
of the cookie  is invalid.***

As workaround, we can use for this scenario the custom context binding I
created [some weeks
ago](http://weblogs.asp.net/cibrax/archive/2008/10/30/rest-and-workflow-services-play-well-together-part-ii.aspx)
to exchange the context information as regular http headers.

\<connectionStrings\>

  \<add name="DurableService" connectionString="Initial
Catalog=SQLWorkflows;Data Source=.\\SQLEXPRESS;Integrated
Security=SSPI;"/\>

\</connectionStrings\>

\<system.serviceModel\>

  \<services\>

    \<service name="ServiceConsole.DurableCalculator"
behaviorConfiguration="MyServiceBehavior"\>

      \<endpoint address="" behaviorConfiguration="MyServiceBehavior"
**binding="webHttpContext"** contract="ServiceConsole.ICalculator" /\>

    \</service\>

\</services\>

\<behaviors\>

  \<endpointBehaviors\>

    \<behavior name="MyServiceBehavior"\>

      \<webHttp /\>

    \</behavior\>

  \</endpointBehaviors\>

  \<serviceBehaviors\>

    \<behavior name="MyServiceBehavior"\>

      \<persistenceProvider
type="System.ServiceModel.Persistence.SqlPersistenceProviderFactory,
System.WorkflowServices, Version=3.5.0.0, Culture=neutral,
PublicKeyToken=31bf3856ad364e35"
connectionStringName="DurableService"/\>

    \</behavior\>

  \</serviceBehaviors\>

\</behaviors\>

\<extensions\>

  \<bindingExtensions\>

    \<add name="webHttpContext"
type="Microsoft.ServiceModel.Samples.WebHttpContextBindingCollectionElement,
WebHttpContext, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null"
/\>

  \</bindingExtensions\>

\</extensions\>

\</system.serviceModel\>

If you pay special attention to the configuration settings above, in
addition to the binding configuration, I also included the
"persitenceProvider" behavior for configuring the persistence provider
that will serialize and store the service instance (In this case, the
SQL server provider).

Now, thanks to this support, my calculator service will survive to
application and server restarts :). Download the complete code from this
[location](/images/legacy/DurableRestCalculator.zip).

