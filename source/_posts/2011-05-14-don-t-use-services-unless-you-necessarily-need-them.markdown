---
layout: post
title: "Don’t use services unless you necessarily need them"
date: 2011-05-14
comments: true
categories: .NET
---

There are multiple factors or requirements that might lead you to
refactor some functionality into services. Here are some examples,

-   You have explicit requirements to distribute work across machines.
    This is a typical example of smart client applications (i.e.
    Silverlight apps), which runs a thin UI layer and have all the
    backend logic as part of services.
-   You need to run code remotely in a specific machine, so a good
    choice is to expose that as a service.  
-   You need to expose certain functionally of your system to other
    parties in a loosely coupled manner. Other parties in this context
    could mean anything such as other applications in the same
    organization, third party client applications, etc.
-   You have some CPU intensive code that you might want to execute in
    another machine to do some load balancing.

If you don’t have any of these requirements, you might want to think
twice before coming up with the crazy idea of building services because
you just think it is cool. The idea of building services must born from
explicit requirements in the system you are creating, and that must be
planned carefully. 

A common problem I usually see is that many developers or architects opt
to build services to follow old principles of work distribution like you
find in N-Tier applications, so they end up with tons of services that
only makes sense in the context of the application they are building,
but not as a unit of reuse.  In many cases, these services are usually
UI driven, which means they are tied to the UI workflow and does not
have a well defined interface.

Here is where I recommend to stick to the Martin Fowler’s [first law of
distribution](http://martinfowler.com/bliki/FirstLaw.html), “Don’t
distribute your objects”. Using a different layer with services for your
business logic to solve scalability issues don’t necessarily makes sense
these days. That’s a problem you can usually solve by scaling
horizontally with additional web servers to support more user requests
at the same time. If you still need to reuse some logic from different
places within your application, nothing prevents you from moving all
that logic to common libraries that run within the same process.

Services when used incorrectly only adds more complexity to the solution
you are creating. You have an additional layer to maintain and
configure, and configuring services is not something trivial. Unless you
don’t use a central repository, you will probably run into a
configuration hell with configuration files everywhere.  This generally
leads to maintainability issues in the long run.

In conclusion, avoid distributing work with services unless you really
need it.

