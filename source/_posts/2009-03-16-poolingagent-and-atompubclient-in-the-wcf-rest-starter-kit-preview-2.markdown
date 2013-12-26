---
layout: post
title: "PollingAgent and AtomPubClient in the WCF Rest Starter kit preview 2"
date: 2009-03-16
comments: true
categories: REST
---

PollingAgent is an utility class for consuming REST services that
implement conditional gets. An instance of this class will periodically
invoke the service within a predefined interval of time, and fire an
event on the client side when a new response is available to consume. It
is internally layered on top of the HttpClient class, so all the
pipeline infrastructure provided by this last one is also supported for
this pooling agent.

As I discussed in the post “[Conditional Get in
REST](http://weblogs.asp.net/cibrax/archive/2008/10/06/importance-of-conditional-gets-in-rest.aspx)”,
conditional gets provide an effective and standard mechanism for caching
information on the client side. It is based on the use of the 
Etag/IfNoneMatch and Last-Modified/IfModifiedSince html headers. This
mechanism is commonly used by the feed readers to be notified about new
items in the feeds.

This pooling agent implementation internally keeps track of those
headers, so it hides many of the conditional gets details from the
client application.

public class PollingAgent : IDisposable

{

  public PollingAgent();

  public HttpClient HttpClient { get; set; }

  public bool IgnoreExpiresHeader { get; set; }

  public bool IgnoreNonOKStatusCodes { get; set; }

  public bool IgnoreSendErrors { get; set; }

  public TimeSpan PollingInterval { get; set; }

  public event EventHandler\<ConditionalGetEventArgs\> ResourceChanged;

  public void Dispose();

  public void StartPolling();

  public void StartPolling(Uri uri);

  public void StartPolling(Uri uri, EntityTag etag, DateTime?
lastModifiedTime);

  public void StopPolling();

}

public class ConditionalGetEventArgs : EventArgs

{

  public ConditionalGetEventArgs();

  public HttpResponseMessage Response { get; set; }

  public Exception SendError { get; set; }

  public bool StopPolling { get; set; }

}

This public API is quite straightforward, a couple of methods to
start/stop polling the REST service, an event “ResourceChanged” to be
notified about changes, and some properties to configure the agent.

AtomPubClient is an specialized version of the HttpClient for consuming
Atom Pub Services. It provides different overloads for creating,
updating, getting or updating existing syndication entries through Atom
pub.

As you can see, the WCF Rest team is definitively doing a great job to
facilitate the adoption of REST services in the platform. 

