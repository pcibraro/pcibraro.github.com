---
layout: post
title: "WCF Workflow services in business applications"
date: 2008-12-03
comments: true
categories: WCF
---

It's pretty interesting to see how useful workflow services can be
sometimes, specially for business applications that require
orchestrating several back-end services calls.

As one of the programmers involved in the development of the WS-I Sample
application for .NET, an application that demonstrates  how to build
interoperable web services in .NET using the WS-I Basic Profile 1.0
(Each vendor in the WS-I working group basically implemented the same
application using its development platform, more information can be
found
[here](http://www.ws-i.org/deliverables/workinggroup.aspx?wg=sampleapps)),
it was a surprise to me to find another implementation as part of the
WCF SDK, made this time all declaratively through WCF workflow services
and just a few activities. To give you an idea, the original version of
the same application took us some days and hundred lines of code just to
have something complete and working.

The SDK version is available under the folder, [SDK
Folder]\\WCF\_WF\_CardSpace\_Samples\\WCF\\Scenario\\WorkflowServices\\Conversations

Workflow services, initially introduced in WCF 3.5 with the Receive and
Send activities, will now take a very important role as part of the Oslo
vision. We will have the opportunity to write pure declarative
long-running services with XAML, including all the WCF contracts for
receiving/sending messages. This will make possible to store the
complete workflow representation in the Oslo repository, and take later
full control of this workflow's instances from Dublin, the new hosting
environment for long-running services. David Chappell has written an
excellent article about WF 4.0, Dublin the oslo vision, you can find it
[here](http://www.davidchappell.com/blog/2008/11/first-look-at-wf-40-dublin-and-oslo.html),
this a must read for developers interested in this area.

