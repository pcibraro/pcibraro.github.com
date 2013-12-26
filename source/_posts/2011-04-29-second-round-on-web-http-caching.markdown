---
layout: post
title: "Second round of Web Http Caching"
date: 2011-04-29
comments: true
categories: .NET
---

As I discussed in my [previous
post](http://weblogs.asp.net/cibrax/archive/2011/04/25/implementing-caching-in-your-wcf-web-apis.aspx),
web caching relies on specific headers that you need to use correctly on
your services. That’s an http application protocol thing, and something
that you can easily use in any application framework that treats Http as
first citizen. This means that you don’t need to implement anything
fancy or exclusively rely on an specific caching technology or
components for doing ouput caching (e.g ASP.NET Cache).

You only need to worry about using the Expires and Cache-Control headers
correctly and optionally setting up a validation process with
conditional gets. If the application framework automatically provides
this for you under the hood, that’s definitely a good thing too.

I promised to write a more detailed post with some extensions for WCF,
but Jose beat me to it. He wrote an [excellent
post](http://jfromaniello.blogspot.com/2011/04/http-cache-and-rest-services.html)
on how to implement some generic extensions for WCF Web Apis, and what’s
more important, how to configure intermediaries like local caches in the
browser or reverse proxies (Squid) to cache the data for us. 

