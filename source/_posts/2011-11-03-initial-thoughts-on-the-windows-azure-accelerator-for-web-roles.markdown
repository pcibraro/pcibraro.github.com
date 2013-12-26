---
layout: post
title: "Initial thoughts on the Windows Azure Accelerator for Web Roles"
date: 2011-11-03
comments: true
categories: .NET
---

Deployment is considered today as one of the major pain points in the
Windows Azure platform.  A simple application deployment today can take
around 20 or 30 minutes to complete, and to make the things even worse,
there is not support for partial updates meaning that a simple change
requires a complete upgrade or deployment.  The Windows Azure team has
recently introduced the support for web deploy in the platform to deal
with this issue. Using Web Deploy, you can deploy a complete application
or part of it in an existing web role instance in a matter of seconds,
which is a huge improvement to what we had so far. 

However, Web Deploy only supports deployments to a single instance in a
web role.  If your web role is made up of two or more instances, which
is the typical configuration instances to be covered by a 100% uptime,
web deploy won’t work in that scenario.

This is where the [Windows Azure Accelerator for Web
Roles](http://waawebroles.codeplex.com/) enters the scene. This is a
project created by the Developer and Platform evangelism team in
Microsoft for deploying one or more websites across multiple Web Role
instances using Web Deploy.  It basically explores the idea of using a
single web role for hosting multiple web sites, simplifying the
deployment and minimizing costs.

As part of the project, you will find an MVC application that you can
deploy in one hosted service in Azure.  That application provides an
visual administrative interface to define and manage the websites or
applications you want to host in the web role. 

As you will be managing multiples websites internally hosted in a web
role, you will need to assign different http host headers to those web
sites and have domains or CName records already set for those. For
example, if the address of your hosted service is cibrax.cloudapp.net
(The MVC application will be listening on this address), you can later
set some CName records in your domain for redirecting users to the
hosted service address ([www.mydomain.com](http://www.mydomain.com) –\>
cibrax.cloudapp.net)

Therefore, you won’t able to use this project if you don’t own a public
domain in which you can set up some CName records.

Another important aspect of the accelerator is that it replicates your
websites across all the instances of the web role breaking that big
limitation in Web Deploy. It internally uses the blob storage to perform
this replication.

In a traditional web application hosted in Azure, you will want to have
some configuration settings as part the hosted service configuration so
they can easily be changed without deploying your application
completely, which is what it would happen if you have everything in the
web.config file for example. This is no longer needed with the
accelerator as you can now deploy an application from scratch in a few
seconds.

