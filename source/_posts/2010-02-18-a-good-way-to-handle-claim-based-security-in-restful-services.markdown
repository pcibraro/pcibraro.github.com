---
layout: post
title: "A good way to handle claim based security in RESTful services"
date: 2010-02-18
comments: true
categories: .NET
---

Dominick just
[blogged](http://www.leastprivilege.com/SecuringWCFDataServicesUsingWIF.aspx)
what I think is one of the best ways to provide claim based security for
RESTful services at the moment. The idea of using simple web tokens for
RESTful services I’ve been in my head for a while, but I was not able to
find enough the time to implement it. Fortunately, Dominick already did
it for us, so it’s really great to have a sample with that.

One piece that is still missing in this architecture is the possibility
to run an in-house STS for issuing simple web tokens rather than using
the ACL service available as part of Windows Azure AppFabric. It would
be great if we could have some built-in support in WIF to issue simple
web tokens, in other words, a built-in security handler for simple web
tokens, and a RESTful endpoint for requesting those. 

The OAuth Wrap specification (the place where the simple web tokens are
defined) also supports the typical ActAs scenario, where a client
application can request a token on behalf of the logged user. This would
be ideal for example, in a web application that want to consume a
RESTful service on behalf of the logged user.    

