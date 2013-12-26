---
layout: post
title: "Visualizing Service Dependencies in SO-Aware"
date: 2010-08-04
comments: true
categories: .NET
---

A common requirement that we received from some customers while we were
in the early design stages of SO-Aware was the ability of tracking
static dependencies between services. For instance, Service A calls
Service B and Service B calls Service X. This feature is not only useful
for documentation but also for helping administrators to determine which
services are going to affected with a change in one of the existing
service. (In that example, a change in the service X would affect
Service A and B).

A service dependency is represented in the repository as a
“ServiceDependency” resource, which contains two simple properties
“ServiceVersion” and “DependantServiceVersion”. Therefore, creating a
new service dependency with the OData API is quite straightforward, and
it’s illustrated in the code above

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: white; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
   1: var serviceDependency = new ServiceDependency
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
   2: {
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: white; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
   3:     ServiceVersion = serviceA,
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
   4:     DependantServiceVersion = serviceB,
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: white; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
   5: }
~~~~

That code is creating a new Dependency resource that represents a
dependency between service B and service A (Service B is using Service
A).

As the OData feed might not be the right thing for visualizing the
dependencies between services, we have also added a nice dependency tree
as part of the web portal to visualize those. So, when you visualize the
dependencies for the service A for instance, you will see something like
this.

![](http://tellagostudios.com/blogEntries/dependencies/Depedencies.png)

