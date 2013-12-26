---
layout: post
title: "My frustrations with MVVM in Silverlight"
date: 2011-03-22
comments: true
categories: .NET
---

There is no doubt that the MVVM pattern offers a clean separation of
concern for building testable user interfaces with WPF and Silverlight. 
This pattern relies on the data binding support in those two
technologies for mapping an existing model class (the view model) to the
different parts of the UI or view.

Someone would say, If you want to develop more testable and maintainable
applications that can evolve in the long run, MVVM is definitely the way
to go in Silverlight, but wait, is that a good decision to make when
something that should take a couple of hours instead takes a day with
MVVM ?.  

In my case, I always want to do the things right, so MVVM was the
approach I decided to use in some outgoing developments with
Silverlight. In used MVVM in the past with WPF with no problems, so I
thought it will be the same thing on Silverlight.

After having worked around six months with the technology, I have to
admit that I got frustrated with many of the limitations of I found for
implementing MVVM with many of the existing controls. In the way I see
it, many of them were definitely developed to support simple RAD
scenarios with code behind, but not for scenarios with data templates
and bindings.   

I wrote a couple of posts in the past with some workarounds to support
data binding in some of the existing controls
([TreeView](http://weblogs.asp.net/cibrax/archive/2011/01/27/workarounds-for-supporting-mvvm-in-the-silverlight-treeview-control.aspx)
and
[ContextMenu](http://weblogs.asp.net/cibrax/archive/2011/01/31/workarounds-for-supporting-mvvm-in-the-silverlight-contextmenu-service.aspx)),
which definitely were not something trivial to find and required
investing many hours of research and testing to understand how the
things worked under the hood. To give an example of how simple things
can become really hard to accomplish with MVVM, I recently came across
this [thread](http://forums.silverlight.net/forums/p/18256/408031.aspx)
about implementing MVVM in the Tab Control while trying to do the same
on my side. Using a converter for binding a model to the control was not
the answer I expected to hear, and sounded like a dirty hack to me. The
same thing can be achieved in WPF with data templates, which is a more
natural way to do things with this model.

The only thing I can say is that MVVM works well for simple scenarios
and common controls like a dropdown list, but you will probably find a
hard time trying to use this pattern in Silverlight unless you know
exactly what the limitations are and the different workarounds you can
use.

