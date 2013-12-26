---
layout: post
title: "Microsoft Synchronization Framework and SSE"
date: 2007-11-22
comments: true
categories: .NET
---

Microsoft has released the first CTP of the synchronization framework,
it is available to download
[here](http://www.microsoft.com/downloads/details.aspx?FamilyId=35E8F16E-AAA4-4919-8B3C-1CE4EA1F6552).

The idea of this framework is to provide a platform or foundation to
synchronize data across multiple data stores through an extensible
architecture based on providers.

This drop comes with three providers out of the box,

​1. Sync provider for ADO.NET, which is the provider for synchronization
across databases  using ADO.NET.

​2. Sync provider for File System, which can be used to synchronize two
file system locations.

​3. Sync provider for SSE (Simple Sharing Extensions) for
synchronization data against SSE feeds. If you are not familiar with
SSE, it is a lightweight protocol initially defined by [Ray
Ozzie](http://rayozzie.spaces.live.com/blog/cns!FB3017FBB9B2E142!285.entry)
that mounts on top of RSS or ATOM to allow for two-way synchronization
among peers. If you are interested in knowing more about this
technology, [Daniel wrote an excellent
post](http://www.clariusconsulting.net/blogs/kzu/archive/2007/06/26/28072.aspx)
in his blog a couple of weeks ago. As Daniel mentioned in his post, we
have been participating of an open-source implementation of SSE,
[SimpleSharing-NET](http://www.codeplex.com/sse/Release/ProjectReleases.aspx?ReleaseId=5301)
that is available to download in CodePlex.

I will be discussing more about the SimpleSharing-NET architecture in
future posts.

