---
layout: post
title: "CRUD Interface for a Service - Is a bad practice ?"
date: 2007-01-25
comments: true
categories: ASP.NET
---

I have been reading in different places that using a CRUD (Create, Read,
Update, Delete) interface for a service is considered a bad practice or
anti-pattern. Therefore, this kind of interface should be avoided at all
cost.

I am not agree with this at all, when I hear the word CRUD, I assume
something like this:

-   CreateCustomer (Create new customer)
-   ModifyCustomer (Modify an existing customer)
-   DeleteCustomer (Delete an existing customer)
-   FindCustomer (Find an existing customer)

This is typical case of a CRUD interface to manage data related to a
customer entity.

What happens if those services represent real business events in the
system that I am trying to model, and the design of them also adheres to
the four tenets of service orientation, is that considered a bad
practice as well ?.

I do not think so.

The sample given in this article ["Principles of Service Design: Service
Patterns and
Anti-Patterns"](http://msdn2.microsoft.com/en-us/library/ms954638.aspx)
 shows an ugly sample of CRUD interface, and it is ok to consider it an
anti-pattern. However, not all the CRUD interfaces are modeled in that
way, in fact, I have never seen such a bad design as the one shown in
that sample.

What do think ?. I would like to hear any opinion about this topic.

