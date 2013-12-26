---
layout: post
title: "Apache Cordova. A new alternative for developing native apps in Win Phone 7"
date: 2012-05-11
comments: true
categories: .NET
---

[Apache Cordova](http://incubator.apache.org/cordova/) is one of those
projects that recently caught my attention for developing applications
in mobile. This project previously known as PhoneGap was donated by
Adobe to the Apache Foundation to be part of a new and attractive open
source alternative for developing mobile applications.  If you haven’t
heard of it before, it basically provides the required infrastructure to
run native applications in different mobile platforms such as IOS,
Android, and Windows Phone 7 using an hybrid approach with an embedded
browser. There is a thin native layer that provides access to different
native features in phone through an standard object model in JavaScript,
so the developers can write their applications using HTML 5 and have
access to different features through that model. Therefore, the two most
important components in this project are the native layer and the
JavaScript component model, which are supported across the different
platforms.

The Microsoft Interoperability team collaborated as part of the project
to support Win7 phone as another available platform, which [was
announced by Abu
Obeida](http://blogs.msdn.com/b/interoperability/archive/2011/12/16/full-support-for-phonegap-on-windows-phone-is-now-complete.aspx)
in December last year. Another major announcement these days was the
support for a [Metro theme in JQuery
Mobile](http://blogs.msdn.com/b/interoperability/archive/2012/04/26/more-news-from-ms-open-tech-announcing-the-open-source-metro-style-theme.aspx),
which gives the ability to give applications written in HTML 5 for this
platform the same look and feel as a native applications.

If you want to know more about the platform, there is an excellent
introductory article written by Colin Eberhardt for the MSDN Magazine,
[“Develop HTML Windows Phone Apps with Apache
Cordova”](http://msdn.microsoft.com/en-us/magazine/hh975345.aspx)

One the things I noticed in the project, is that there is available a
project template in Visual Studio for creating the initial application.
However, you need to manually deploy that template in the right folder
to make it available in Visual Studio, which might be an error prone
task. I decided to make a little contribution to the project and write
an Visual Studio extension (a vsix package) to automatically deploy the
template, which I am not sure yet if it is going to be approved or not,
but I think it is a nice thing to have for easing adoption. This is part
of the [fork associated to my
user](https://github.com/pcibraro/incubator-cordova-wp7) in case you
want to use it.

