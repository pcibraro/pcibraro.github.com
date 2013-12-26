---
layout: post
title: "The rise of Single Page Applications (SPA) in Web Mobile"
date: 2012-01-27
comments: true
categories: .NET
---

Limited connectivity is one of the main challenges in web mobile. The
stateless nature of http causes that  content  and associated static
files like scripts or images be transmitted over the wire every time a
page is fully refreshed (assuming http caching is not implemented
correctly). Diverse techniques have emerged over the years to solve part
of that problem by using the browser AJAX support. One of these
techniques, which drastically changed the way we develop web
applications is know as single page applications.

A single page application design assumes that good part of the website
layout is fixed and there are a few placeholders for showing dynamic
content. The layout and associated files can be download once, and the
rest can be dynamically changed using AJAX calls to a backend Web
API.    

There are also a few improvements that can be made like caching all the
static content and associated files, and use client templates to remove
all the overhead involved with downloading html markup on every AJAX
call. By using client templates (e.g. JQuery templates, JSRender,
Mustache) , only the data (usually JSON data) is negotiated with the
backend, and the templates can be cached as static content as well.

In the area of web mobile, it is pretty important to optimize the use of
the available bandwidth to support scenarios where the connectivity is
very unreliable or limited data plans are used. This is exactly where
single page applications make a lot of sense. As this kind of
application makes a heavy use of javascript for DOM manipulation, it
might not be ideal for all kind of mobile devices (the device should
support DOM manipulation and XHR).

I personally think this kind of architecture for a web application will
become a common norm now that we have devices with web support
everywhere.

