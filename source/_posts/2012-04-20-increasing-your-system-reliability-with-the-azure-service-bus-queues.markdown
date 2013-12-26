---
layout: post
title: "Increasing your system reliability with the Azure Service Bus Queues"
date: 2012-04-20
comments: true
categories: .NET
---

A common scenario for many web applications running in the cloud is to
integrate with existing systems through web services (no matter the
messaging style they use). Although in these scenarios, an SLA is
typically used as an agreement between the two parties to assure certain
level of availability, many things can still fail. Therefore, it is
always a good idea to have a mechanism in place to handle any possible
error condition and retry the execution when it is possible.

As example, you could have a web application that calls an online CRM
system (like Salesforce.com or MS Dynamics) for allowing the users to
report incidents. In that scenario, we can not assume any possible call
to the CRM system will always succeed. On the other hand, this kind of
call does not require an immediate response for the user, so it can be
scheduled for later execution and retried if something unexpected
happens.

A persistent storage for the messages like a queue is always a good
choice for decoupling clients from services, and also accomplish the
goal previously discussed. When you move to Windows Azure, there are two
different queue offerings, Queues as part of the Storage service, and
Queues (or Topics) as part of the Service Bus.

The Service Bus SDK provides a programming model on top of WCF for
consuming or sending messages to the queues, which makes this solution
very appealing for this scenario. The Service Bus also provides Topics,
which are an specific kind of queues that supports subscriptions.

By using Service Bus Queues, the calls to any third party service could
be eventually wrapped up in WCF services that are invoked by the web
application. The following image illustrates the possible architecture,

[![queues](http://weblogs.asp.net/blogs/cibrax/queues_thumb_6A33755A.jpg "queues")](http://weblogs.asp.net/blogs/cibrax/queues_131E8A64.jpg)

One of the core classes in the WCF programming model for the Service Bus
Queues is BrokeredMessage. This class represents a message that can be
sent to/from an existing queue, and contains a lot of  standard
properties such as MessageId, ReplyTo, SessionId or Label to name a few.
It also contain a dictionary for custom properties that an application
can assign to the message.

In the example above, the MessageId property can be used to correlate
the input and out messages in the queue and update the corresponding
result in the web application. The WCF service can also use the ReplyTo
property, which represents the queue name in which the response should
go. Sessions is another concept supported in the Service Bus Queues,
which are useful for correlating a set of messages as a batch for
processing.   This is something optional and requires the queue to
support sessions when they are created.

In WCF, the BrokeredMessage is assigned to the service call with a
message property BrokeredMessageProperty as it is illustrated bellow,

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
var channelFactory = new ChannelFactory<ICRMServiceClient>("crminput");
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
var clientChannel = channelFactory.CreateChannel();
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
 
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
// Use the OperationContextScope to create a block within which to access the current OperationScope
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
using (var scope = new OperationContextScope((IContextChannel)clientChannel))
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
{
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
    // Create a new BrokeredMessageProperty object
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
    var property = new BrokeredMessageProperty();
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
 
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
    // Use the BrokeredMessageProperty object to set the BrokeredMessage properties
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
    property.Label = "Incident";
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
    property.MessageId = Guid.NewGuid().ToString();
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
    property.ReplyTo = "sb://xxx.servicebus.windows.net/input";
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
 
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
    // Add BrokeredMessageProperty to the OutgoingMessageProperties bag provided 
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
    // by the current Operation Context 
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
    OperationContext.Current.OutgoingMessageProperties.Add(BrokeredMessageProperty.Name, property);
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
    
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
    //Do the service call here
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
 
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
 
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
}
~~~~

On the service side, the BrokeredMessageProperty instance can be
retrieved in a similar way

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
var incomingProperties = OperationContext.Current.IncomingMessageProperties;
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
var property = incomingProperties[BrokeredMessageProperty.Name] as BrokeredMessageProperty;
~~~~

Another important feature in the programming model for supporting
execution retries in our example is the ReceiveContext. By decorating
the WCF service contract with the ReceiveContextEnabled attribute, we
can manually specify whether the operation was successfully completed or
not. If the operation was not completed, the message will remain in the
queue and the operation will be executed again next time.

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
[ServiceContract()]
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
public interface ICRMService
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
{
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
    [OperationContract(IsOneWay=true)]
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
    [ReceiveContextEnabled(ManualControl = true)]
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
    void CreateCustomer(Customer customer);
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
}
~~~~

 

The following code shows how the operation implementation looks like,

 

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
var incomingProperties = OperationContext.Current.IncomingMessageProperties;
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
var property = incomingProperties[BrokeredMessageProperty.Name] as BrokeredMessageProperty;
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
 
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
//Complete the Message
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
ReceiveContext receiveContext;
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
if (ReceiveContext.TryGet(incomingProperties, out receiveContext))
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
{
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
   //Do Something                
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
   receiveContext.Complete(TimeSpan.FromSeconds(10.0d));
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
}
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
else
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
{
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
   throw new InvalidOperationException("...");
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
}
~~~~

As you can see, the ReceiveContext instance is marked as complete or
implicitly set as not complete when an exception is thrown.

The assemblies for the Service Bus SDK are available as part of a Nuget
package “Windows Azure Service Bus”. As part of the package
registration, all the required configuration extensions for the WCF such
as custom bindings and behaviors are also added in the application
configuration file.

NetMessagingBinding is the one you need to use for sending or receiving
messages from a queue. That binding is constantly polling the queues for
detecting new messages, and activating the WCF service when a new
message arrives. For that reason, you need to keep the web application
App pool running all the time. This can be accomplished in Windows Azure
with a simple approach as this one mentioned by Christian Weyer in this
[post](http://weblogs.thinktecture.com/cweyer/2011/01/poor-mans-approach-to-application-pool-warm-up-for-iis-in-a-windows-azure-web-role.html).

