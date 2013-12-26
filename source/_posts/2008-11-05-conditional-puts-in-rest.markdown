---
layout: post
title: "Conditional Puts in REST"
date: 2008-11-05
comments: true
categories: REST
---

Conditional puts is a technique generally used in a REST architecture to
inform clients about possible conflicts when multiple updates are
performed simultaneously over the same resource version. It basically
works as fist-write / first-win approach, a client can only commit an
operation only if the underline resource has not changed in the
meanwhile, otherwise it may receive a http conflict error.

As the "[conditional
get](http://weblogs.asp.net/cibrax/archive/2008/10/06/importance-of-conditional-gets-in-rest.aspx)"
approach I discussed some weeks ago, conditional puts are also based on
the use of the headers "Last-Modified" and "Etag" (the If-Not-Modified
and If-None-Match request headers are their flip side). It is used
nowadays for example by the "[ATOM publishing
protocol](http://en.wikipedia.org/wiki/ATOM)", and there are some plans
to support it in [Amazon S3](http://aws.amazon.com/s3/).

Let's see how this approach works with a simple example of two clients
(A and B) trying to update a resource R1.

​1. Client A performs a GET over R1 (Version1). The response given by
the REST service will include the resource representation and a header
"Etag" with the resource version, "1" in this case (Last-Modified could
also be used).

​2. Client B performs a GET over R1 (Version1). It gets the same
representation as the client A

​3. Client B modifies R1 and invokes a PUT on the server. This
invocation includes the modified version of the resource and a header
"If-None-Match" with the current resource version (Version 1). As result
of this modification, the server returns a Http OK and increments the
resource version by one (Version 2).

​4. Client A modifies R1 and try to invoke a PUT on the server. (This
invocation also includes a "If-None-Match" header with the resource
version "Version 1"). The service detects that the resource changed
since the last get, so it throws a conflict error.

If you want to know more about how to implement this technique with WCF,
the new WCF REST Starter Kit includes a complete example "ConditionPut"
that illustrates all the steps involved in the implementation of this
scenario.

