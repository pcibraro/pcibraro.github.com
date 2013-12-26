---
layout: post
title: "Caching strategies for SOAP and REST Services"
date: 2011-05-18
comments: true
categories: .NET
---

SOAP services are in nature transport agnostics so they can not rely on
specific transport features. Http is a great example where SOAP services
make a poor use of Http as application protocol. This means that many of
the http constraints are simply ignored, http headers are not used at
all, messages are not self descriptive (You can not easily infer what a
message does by looking at the content), the uniform interface is not
used either as everything goes as a POST to the server and the list
keeps growing. This makes impossible to leverage the existing web
architecture and use intermediaries for caching results.  

As consequence of this, the only viable alternative is to provide
caching as part of the service implementation. Caching at this level
might be a good option for improving the service performance when some
bottlenecks are detected and caused by IO operations with long delays
(Calls to databases, legacy services, etc.) or CPU intensive code. In
both scenarios, you might want to keep the results in memory as much as
you can to reuse them in the service implementation. Here is where a
distributed cache solution like AppFabric caching, NCache or MemCached
makes a lot of sense. Local caches in memory such as Enterprise library
might be another option but only if you are not running services in a
farm scenario.

Client applications might opt to cache responses from the services too,
but there is not any explicit mechanism for negotiating the lifetime of
the cached copies between clients and services, making necessary to use
out of band information.

[![Cache](http://weblogs.asp.net/blogs/cibrax/Cache_thumb_6BAE5B03.jpg "Cache")](http://weblogs.asp.net/blogs/cibrax/Cache_663FEA5F.jpg) 

On the other hand, REST services can reuse all the available caching
infrastructure in the web by leveraging Http as application protocol.
Http already specifies headers for caching control and a process to
revalidate cached copies that any intermediary can use (No need to use
out of band information). Intermediaries in this contexts are usually
represented by local caches like the one might find in a browser,
proxies or reverse proxies. In described the implementation details
[here](http://weblogs.asp.net/cibrax/archive/2011/04/25/implementing-caching-in-your-wcf-web-apis.aspx).

[![Cache2](http://weblogs.asp.net/blogs/cibrax/Cache2_thumb_0867D019.jpg "Cache2")](http://weblogs.asp.net/blogs/cibrax/Cache2_02F95F75.jpg)

REST services can still use a caching layer on the service
implementation as it was discussed with SOAP services or use some output
caching mechanism in the web server (ASP.NET Cache for example), but a
good thing is that the web can be your friend in this matter too.

