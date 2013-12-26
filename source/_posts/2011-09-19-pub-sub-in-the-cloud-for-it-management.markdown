---
layout: post
title: "Pub/Sub in the cloud for IT Management"
date: 2011-09-19
comments: true
categories: .NET
---

In the recent “Build” event, Microsoft introduced a new feature Windows
Server 8 known as “Windows Powershell Web Access” for exposing a the
server powershell console through a web interface. Although this feature
looks very promising in first place, I only think it is convenient for
intranet scenarios. I don’t initially see an organization exposing their
servers directly to internet for using this feature from a phone as it
represents a high risk.

As I discussed a time ago in this post [“Pub/Sub in the cloud
..”](http://weblogs.asp.net/cibrax/archive/2011/06/06/pub-sub-in-the-cloud-a-brief-comparison-between-azure-service-bus-and-pubnub.aspx),
technologies like Windows Azure Service Bus, PubNub or Fanout 
(implemented in Node.JS) can act as intermediaries in the cloud for
passing messages between clients and servers that are not publically
addressable or can talk each other directly in the same network. All
these technologies relies on outbound http connections for talking to
the cloud, which are not typically blocked by firewalls.

What happens if you decide to extend this idea of passing messages from
clients to servers to the IT Management world, and those messages
actually represent actions you want to perform on your servers like
restarting a web server or getting a list of active process. You would
be able to control any of the servers in your organization from any
device connected to the internet right ?. Do you think it’s too risky
too ?.  Ok, let’s add an additional layer on the cloud, which can act as
firewall and control access about who can do what on the different
servers. This additional layer on the cloud can also have a very nice
HTML 5 visual interface that any IT guy in your organization can easily
understand and use from any browser or device connected to the internet.
At that point, you have everything you need to be in control of your
servers from anywhere, even when you are driving back from work.

That’s the main idea of project where I have been working for the last
few months in Tellago Studios and it was announced to the world today as
[Moesion.com](http://moesion.com). Take a look and subscribe to the beta
program today, there will be a free personal edition that you could use
for managing your servers.

