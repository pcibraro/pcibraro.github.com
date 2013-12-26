---
layout: post
title: "Extending the SO-Aware repository with custom metadata"
date: 2011-05-25
comments: true
categories: .NET
---

One of the main features that SO-Aware provides is the central
repository for storing service artifacts (WSLD, schemas, bindings) and
configuration that any organization generates. This central repository
is completely exposed as an OData service that third party applications
and tools can easily consume using Http.

However, the initial SO-Aware release lack of support for extending the
standard artifacts with custom metadata. For example, if you wanted to
associate some external documentation to your WCF bindings or set the
developer or owner for an specific service,  that was not supported out
of the box.

Based on all the feedback we received from customers, we decided to
include this feature in the latest version. From the UI standpoint, you
will see a new tab “Custom Metadata” in every resource.

[![Metadata](http://weblogs.asp.net/blogs/cibrax/Metadata_thumb_3CBFAD4B.png "Metadata")](http://weblogs.asp.net/blogs/cibrax/Metadata_12A02530.png)     

The custom metadata that you can associate to existing resources is
represented by key/value pairs (Metadata Term and Value). For example,
the Metadata term can be “Owner” and the value “Pablo Cibraro”.

SO-Aware will create also a dictionary of available “Metadata” terms
that every user can reuse in the repository.  These metadata terms can
also be queried and modified through the standard OData API that the
repository exposes. 

Enjoy!!.

