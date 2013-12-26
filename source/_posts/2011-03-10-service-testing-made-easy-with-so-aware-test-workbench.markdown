---
layout: post
title: "Service Testing made easy with SO-Aware Test Workbench"
date: 2011-03-10
comments: true
categories: .NET
---

I happy to announce today a new addition to our SO-Aware service
repository toolset, SO-Aware Test Workbench, a WPF desktop application
for doing functional and load testing against existing WCF Services.

This tool is completely integrated to the SO-Aware service repository,
which makes configuring new load and functional tests for WCF Soap and
REST services a breeze. From now on, the service repository can play a
very important role in an organization by facilitating collaboration
between developers and testers.

[![testing\_tool](http://weblogs.asp.net/blogs/cibrax/testing_tool_thumb_1BADC864.jpg "testing_tool")](http://weblogs.asp.net/blogs/cibrax/testing_tool_59D708E2.jpg)

Developers can create and register new services in the repository with
all the related artifacts like configuration. On the other hand, Testers
can just pick one of the existing services in the repository and create
functional or load tests from there, with no need to deal with specific
details of the service implementation, location or configuration
settings. Developers and Testers can later use the result of those tests
to modify the services or adjust different settings on the tests or
service configuration.

Gustavo Machado, one of the developers behind this project, has written
an [excellent
post](http://thegsharp.wordpress.com/2011/03/10/wcf-load-testing-with-so-aware-test-workbench/)
describing all the functionality that can find today in the tool. You
can also see the tool in action in this [Endpoint Tv
episode](http://channel9.msdn.com/Shows/Endpoint/endpointtv-Performance-Testing-with-SO-Aware)
with Jesus and Ron Jacobs.

