---
layout: post
title: "REST and Workflow services play well together - Part II"
date: 2008-10-30
comments: true
categories: REST
---

In my previous post ["REST and Workflow services play well
together"](http://weblogs.asp.net/cibrax/archive/2008/10/24/rest-and-workflow-services-play-well-together.aspx),
I mentioned that Http Cookies were one of the built-in mechanisms for
transferring the workflow context across calls between the client and
the service. While cookies work well for http services, from my point of
view, simple Http Headers naturally fit better in a REST architecture.
As consequence, I decided to extend the WCF channel stack to support a
new a custom channel (Or custom binding) for transferring the workflow
context as a http header.

The configuration of this binding WebHttpContextBinding is quite
straightforward, I just created a new binding "WebHttpContextBinding"
that can easily configured for workflow service,

\<services\>

  \<service name="RestWorkflows.OrderWorkflow"\>

    \<endpoint address="" behaviorConfiguration="MyServiceBehavior"
binding="webHttpContext" bindingConfiguration="myServiceBinding"
contract="RestWorkflows.IOrderService" /\>

  \</service\>

\</services\>

\<bindings\>

  \<webHttpContext\>

    \<binding name="myServiceBinding"\>\</binding\>

  \</webHttpContext\>

\</bindings\>

\<behaviors\>

  \<endpointBehaviors\>

     \<behavior name="MyServiceBehavior"\>

       \<webHttp /\>

     \</behavior\>

  \</endpointBehaviors\>

\</behaviors\>

\<extensions\>

  \<bindingExtensions\>

    \<add name="webHttpContext"
type="Microsoft.ServiceModel.Samples.WebHttpContextBindingCollectionElement,
WebHttpContext, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null"
/\>

  \</bindingExtensions\>

\</extensions\>

The result of using this binding will be a new Http Header WscContext
(encoded as Base64) being transmitted between the client and the
service,

\<MessageLogTraceRecord Time="2008-10-29T14:22:59.5258021-02:00"
Source="TransportSend"
Type="System.ServiceModel.Channels.BodyWriterMessage"
xmlns="[http://schemas.microsoft.com/2004/06/ServiceModel/Management/MessageTrace"](http://schemas.microsoft.com/2004/06/ServiceModel/Management/MessageTrace%22)\>\
   \<Addressing\>\
     \<Action\>\</Action\>\
   \</Addressing\>\
   \<HttpResponse\>\
      \<StatusCode\>Created\</StatusCode\>\
      \<WebHeaders\>     
**\<WscContext\>77u/PENvbnRleHQgeG1sbnM9Imh0dHA6Ly9zY2hlbWFzLm1pY3Jvc29mdC5jb20vd3MvMjAwNi8wNS9jb250ZXh0Ij48UHJvcGVydHkgbmFtZT0iaW5zdGFuY2VJZCI+NDZkZGY3NTktMzdjNS00ZDZlLWI2ZTItNjVmMDNjMDVmYTAyPC9Qcm9wZXJ0eT48L0NvbnRleHQ+\</WscContext\>\
**       \</WebHeaders\>\
    \</HttpResponse\>\
    \<order
xmlns="[http://starbucks.example.org"](http://starbucks.example.org%22/)
xmlns:i="[http://www.w3.org/2001/XMLSchema-instance"](http://www.w3.org/2001/XMLSchema-instance%22)\>\
      \<cost\>6\</cost\>\
      \<drink\>latte\</drink\>\
      \<next
xmlns:a="[http://example.org/state-machine"](http://example.org/state-machine%22)\>\
         \<a:next\>

         
\<a:rel\>[http://starbucks.example.org/payment](http://starbucks.example.org/payment)\</a:rel\>\
           \<a:type\>application/xml\</a:type\>\
          
\<a:uri\>[http://localhost:8000/payment/order/0b756654-161b-43d0-912f-4236552dc337](http://localhost:8000/payment/order/0b756654-161b-43d0-912f-4236552dc337)\</a:uri\>\
         \</a:next\>\
         \<a:next\>\
            
\<a:rel\>[http://starbucks.example/order/update](http://starbucks.example/order/update)\</a:rel\>\
             \<a:type\>application/xml\</a:type\>\
            
\<a:uri\>[http://localhost:8000/order/0b756654-161b-43d0-912f-4236552dc337](http://localhost:8000/order/0b756654-161b-43d0-912f-4236552dc337)\</a:uri\>\
         \</a:next\>\
       \</next\>\
     \</order\>\
\</MessageLogTraceRecord\>

\<MessageLogTraceRecord Time="2008-10-29T14:22:59.5726021-02:00"
Source="TransportReceive"
Type="System.ServiceModel.Channels.BufferedMessage"
xmlns="[http://schemas.microsoft.com/2004/06/ServiceModel/Management/MessageTrace"](http://schemas.microsoft.com/2004/06/ServiceModel/Management/MessageTrace%22)\>\
   \<HttpRequest\>\
      \<Method\>PUT\</Method\>\
       \<QueryString\>\</QueryString\>\
       \<WebHeaders\>\
     
**\<WscContext\>77u/PENvbnRleHQgeG1sbnM9Imh0dHA6Ly9zY2hlbWFzLm1pY3Jvc29mdC5jb20vd3MvMjAwNi8wNS9jb250ZXh0Ij48UHJvcGVydHkgbmFtZT0iaW5zdGFuY2VJZCI+NDZkZGY3NTktMzdjNS00ZDZlLWI2ZTItNjVmMDNjMDVmYTAyPC9Qcm9wZXJ0eT48L0NvbnRleHQ+\</WscContext\>\
**        \<Content-Length\>566\</Content-Length\>\
        \<Content-Type\>application/xml\</Content-Type\>\
        \<Expect\>100-continue\</Expect\>\
         \<Host\>localhost:8000\</Host\>\
      \</WebHeaders\>\
    \</HttpRequest\>\
    \<order
xmlns="[http://starbucks.example.org"](http://starbucks.example.org%22/)
xmlns:i="[http://www.w3.org/2001/XMLSchema-instance"](http://www.w3.org/2001/XMLSchema-instance%22)\>\
          \<cost\>6\</cost\>\
          \<drink\>cappuchino\</drink\>\
          \<next
xmlns:a="[http://example.org/state-machine"](http://example.org/state-machine%22)\>\
            \<a:next\>\
             
\<a:rel\>[http://starbucks.example.org/payment](http://starbucks.example.org/payment)\</a:rel\>\
              \<a:type\>application/xml\</a:type\>\
             
\<a:uri\>[http://localhost:8000/payment/order/0b756654-161b-43d0-912f-4236552dc337](http://localhost:8000/payment/order/0b756654-161b-43d0-912f-4236552dc337)\</a:uri\>\
             \</a:next\>\
             \<a:next\>\
              
\<a:rel\>[http://starbucks.example/order/update](http://starbucks.example/order/update)\</a:rel\>\
               \<a:type\>application/xml\</a:type\>\
              
\<a:uri\>[http://localhost:8000/order/0b756654-161b-43d0-912f-4236552dc337](http://localhost:8000/order/0b756654-161b-43d0-912f-4236552dc337)\</a:uri\>\
             \</a:next\>\
           \</next\>\
       \</order\>\
\</MessageLogTraceRecord\>

The code for the custom binding is available to download from [this
location](/images/legacy/RestWorkflowsII.zip).

