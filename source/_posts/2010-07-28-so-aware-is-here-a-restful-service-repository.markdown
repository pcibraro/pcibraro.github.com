---
layout: post
title: "SO-Aware is here. A RESTful service repository"
date: 2010-07-28
comments: true
categories: .NET
---

I am very proud to announce today the release of our product in [Tellago
Studios](http://tellagostudios.com),
[SO-Aware](http://tellagostudios.com/products/so-aware%E2%84%A2), a
RESTful service repository based on OData.  The main difference between
SO-Aware and other existing products is that everything is exposed as
resources via OData feeds that can be retrieved or manipulated with
standard http verbs like POST, PUT, GET or DELETE (All this is
implemented with WCF data services).

To give a good example of what you can do in SO-Aware, all the
integration tests that you define for your services are also exposed as
http resources, so you can execute a test and get the results with a
simple http request, making the integration with any existing build
management system or testing framework very simple.

In addition, SO-Aware is closely integrated with WCF by giving access to
resources like Binding or Behaviors that you can associate to your
existing services for having a centralized configuration repository, and
making service testing much easier too. In this sense, SO-Aware provides
an specialized API for configuring WCF clients or services from the
central repository with just a few lines of code that removes completely
most of the common deployment pains with configuration files.

You can find a more detailed description of the product and its features
[here](http://tellagostudios.com/products/so-aware%E2%84%A2).

I will be discussing many of the cool features that you can find in
SO-Aware in the next posts, so stay tuned.     

