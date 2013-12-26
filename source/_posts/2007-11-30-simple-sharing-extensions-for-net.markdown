---
layout: post
title: "Simple Sharing Extensions for .NET"
date: 2007-11-30
comments: true
categories: .NET
---

As I mentioned in a previous post, I have been very busy lately working
with [Daniel](http://weblogs.asp.net/cazzu),
[Mariano](http://weblogs.asp.net/marianor/) and other guys here in
Clarius on an open-source implementation of [Simple Sharing Extensions
for .NET](http://www.codeplex.com/sse). The architecture of the project
is still evolving, many changes have been introduced since the the first
release, and it will continue this way as long as we find things to
improve or new features are added. For this reason is that your feedback
is quite important for us at this point.

 

![](/images/legacy/SSE_Framework.jpg)

 

As the diagram above shows, we basically have three main parts or
components that conforms to our architecture:

​1. SSE Model Classes: They represent the classes needed to parse and
maintain the SSE metadata. In this group, you can find classes such as
Sync to represent the synchronization history, XmlItem that contains the
item xml payload or Item, which is a combination of a payload with sync
metadata.

​2. Sync Engine: It represents the synchronization engine that process
and applies the SSE rules to the items at runtime. It basically receives
items from two different sources or repositories and makes a complete
merge, determining new item additions, deletions, modifications or
conflicts, when an item was modified in both sources according to the
SSE rules.

​3. Repositories: This is the part of the architecture that can be
extended, and where the developers can have an opportunity to get their
hands dirty. In a few words, a repository represents a source of Items
that contains an xml payload and sync metadata. It is responsible to
provide and maintain that information in a underline storage, which can
be memory, files, databases, or any other kind of storage. Here, you can
also find an specific kind of repository, a CompoundRepository, which
implements the Repository API and delegates the work to two new classes,
XmlRepository and SyncRepository. The first one is only responsible for
maintaining application specific data or payloads, and the second one
for the sync metadata, the CompoundRepository then, knows how to
correlate both parts (the payload and the sync info) through the SSE
identifier that they have in common.

So, depending on the sync scenario, you can just implement a part of a
repository (Deriving from XmlRepository or SyncRepository) or a complete
repository (Deriving from Repository).

For the moment, there is only a SyncRepository that keeps the sync
metadata in a database (Access, SQL Server or SQL CE), but more
repositories will be available soon.

Just to have an idea of how a synchronization scenario looks like, we
could have for example two repositories, one that knows to how to read
and maintain information about outlook contacts and another repository 
that knows how to publish and retrieve items from the Microsoft Live
Personal SSE service
([http://sse.mslivelabs.com/](http://sse.mslivelabs.com/)). This will
allow us a bidirectional synchronization of outlook contacts using
perhaps the SSE service only as a relay agent.

![](/images/legacy/SSE_Scenario.jpg)

If you are interested in implementing any kind of synchronization
scenario, go ahead and take a look at this project :-).

