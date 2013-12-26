---
layout: post
title: "Unit Tests for WCF (and Moq) Part II"
date: 2008-05-20
comments: true
categories: Moq
---

 

My latest post about "creating wrapper classes" for mocking the WCF
context has started generating some
[controversies](http://weblogs.asp.net/rosherove/archive/2008/05/19/two-faced-commits-how-the-alt-net-community-is-becoming-more-and-more-dogmatic.aspx).
I initially wrote those classes to decouple my service implementation
from the concrete implementation of the WCF context,  something that 
did not have anything to do with the static nature of the context. (I
can still pass the current instance as an argument to my service
implementation).

As I mentioned in my previous post, the WCF context is neither a base
class (with virtual methods) nor an interface, so it can not easily
mocked, that is main issue here.

I am not sure if TypeMock allows mocking this kind of class as well,
probably yes, I have never used that tool before so I do not know.  It
would be great to have a natural or easy support in C\# for mocking
non-virtual methods or statics, but it is not there now. I personally
prefer writing some simple wrapper classes or ["utterly useless
wrappers" as someone call
them](http://www.elilopian.com/2008/05/19/mocking-frameworks-dream-feature/) 
(In fact, it only took me 5 minutes to write them, and I do not think my
productive was affected by that) rather than creating code that can only
be tested by an specific tool. I am not saying that you should not use
TypeMock, anyone is free to use the tool that works best for his needs.

