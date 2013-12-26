---
layout: post
title: "Disposing a WCF Proxy"
date: 2009-06-26
comments: true
categories: .NET
---

A common misconception is to think that a WCF proxy can be used and
disposed as any other regular class that implements IDisposable.
However, the IDisposable implementation in the WCF client channel is not
like any other, and it can throws exceptions. A phrase I got from [this
thread](http://weblogs.asp.net/controlpanel/blogs/this%20discussion%20on%20the%20WCF%20forum)
in the WCF forum will give you a better context about this scenario,

*“I think a key thing to notice is that Close() often implies doing
"real work" that may fail, including network communication handshakes to
shutdown sessions, committing transactions, etc.”*

As consequence of this, we should add some code to handle any possible
exception for network errors when a proxy is closed/disposed.

The common pattern for cleaning up a proxy is the following,

try \
{ \
... \
client.Close(); \
} \
catch (CommunicationException e) \
{ \
... \
client.Abort(); \
} \
catch (TimeoutException e) \
{ \
... \
client.Abort(); \
} \
catch (Exception e) \
{ \
... \
client.Abort(); \
throw; \
} \

The problem is, I do not want to have that code everywhere in my
application. A good solution for that is to use a helper class like
[this
one](http://bloggingabout.net/blogs/erwyn/archive/2006/12/09/WCF-Service-Proxy-Helper.aspx).

[Michelle](http://www.dasblonde.net/) recently announced a new project
that automatically generates a proxy helper class like that in Visual
Studio.
[http://wcfproxygenerator.codeplex.com/](http://wcfproxygenerator.codeplex.com/)

That is a feature I would like to see out of the box in the next version
of WCF :).

