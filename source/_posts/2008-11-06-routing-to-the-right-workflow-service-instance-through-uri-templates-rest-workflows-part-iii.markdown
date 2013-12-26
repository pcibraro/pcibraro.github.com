---
layout: post
title: "Routing to the right workflow service instance through URI templates (REST workflows part III)"
date: 2008-11-06
comments: true
categories: REST
---

Continuing from my previous posts "[REST and Workflow services play well
together
I](http://weblogs.asp.net/cibrax/archive/2008/10/24/rest-and-workflow-services-play-well-together.aspx)"
and "[REST and Workflow services play well together
II](http://weblogs.asp.net/cibrax/archive/2008/10/30/rest-and-workflow-services-play-well-together-part-ii.aspx)",
in which I created a new binding to send the workflow context as an http
header, I've applied some modifications to that sample to support also
routing through URI templates. This is very helpful for scenarios where
client applications are not necessarily aware of the existence of a
workflow services on the other end, so the context information is
reconstructed from the resource URI using Uri templates.

For example, the following Uri template "/orders/{instanceId}" applied
to the Uri
[http://localhost:8000/orders/3C3DE415-71F4-4538-ADA6-87A735827DF6](http://localhost:8000/orders/3C3DE415-71F4-4538-ADA6-87A735827DF6)
will result in a match for the key "instanceId" with a value of
"3C3DE415-71F4-4538-ADA6-87A735827DF6". We can now pass that instanceId
variable to the WF context correlation manager so it will be able to
locate and load the corresponding workflow service instance.

The configuration for the binding on the service side looks as follow,

\<webHttpContext\>

\<binding name="myServiceBinding" contextMode="UriTemplate"\>

  \<uriTemplates\>

    \<add name="orders" value="order/{instanceId}"\>\</add\>

    \<add name="payments" value="payment/order/{instanceId}"\>\</add\>

  \</uriTemplates\>

\</binding\>

\</webHttpContext\>

UriTemplates is a collection, you can configure there all the possible
URIs that your workflow can handle. In that example, instanceId is the
name of the variable used by WF to locate the workflow instance, other
variable names can also be used. For example "conversationId" can be
used to continue the workflow in a specific ReceiveActivity (Usually
required when the workflow is waiting for some event in a parallel
branch activity)

We can now return the following links (representing the possible
branches in the workflow) to the client,

currentOrder.Next = new Next[] {

    new Next

    {

        Rel = "http://starbucks.example.org/payment",

        Uri = "http://localhost:8000/payment/order/" +
WorkflowEnvironment.WorkflowInstanceId.ToString(),

    },

    new Next

    {

        Rel = "http://starbucks.example/order/update",

        Uri = "http://localhost:8000/order/" +
WorkflowEnvironment.WorkflowInstanceId.ToString()

    }

};

The client application will not need to remember to send a context
header in the next execution to resume an existing workflow instance,
the Uri will be enough to parse the context information.

Download the complete sample from [this
location](/images/legacy/RestWorkflowsIII.zip)

