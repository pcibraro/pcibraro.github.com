---
layout: post
title: "Using Windows Azure Regions efficiently"
date: 2011-10-26
comments: true
categories: .NET
---

Moving your application to the cloud might not be easy as it sounds. The
typical sample we always see in documentation or demos about an ASP.NET
application created from the Visual Studio template and deploy it in
Azure as a package is definitely very far from reality. There are
multiple factors that can affect the response time and availability of
your applications in the cloud but you can not easily see until you
embark on a real project. Application distribution and deployment is one
of those factors, and the one we are going to discuss in this post.

Windows Azure uses the idea of regions to manage the physical
distribution of your applications and data. A region in a nutshell
represents a Microsoft data center where you can deploy your application
or part of it. Nowadays, you can find multiple regions spread across the
globe in places such as North America, Europe and Asia.

There is another abstraction layer on top of regions called "Affinity
Groups", which simply tells the Fabric controller to do its best to
ensure groups of related services are deployed in close proximity
whenever possible to ensure optimization for inter-app communication.
For example, in a typical web application with a database,  you will
want to deploy the web application as close as possible to the database
server. 

Finally, another important concept you need to understand in this
deployment model is the idea of “hosted service”. A hosted service
basically represents an unit of deployment associated to a public DNS in
the cloud. You deploy your applications in a hosted service, which is
also tied to a region or affinity group. What Azure gives you is a
public address for reaching that hosted service in the cloud, which is
“[you app name].cloudapp.net” for applications hosted in the production
environment and an auto generated guid for applications hosted in the
staging environment. You can also think in a hosted service as a load
balancer that forwards requests from that public address to one of the
instances or VMs associated to it.

Let’s discuss the aspects you need to consider for selecting the right
regions for deploying your application.

-   Network latency: reducing the network latency is very important as
    it will impact directly on the response time for your applications.
    You will want to have your users as close as possible physically
    from your applications and data. For example, if you have some
    potential users in the US and Europe, you will want to deploy two
    exact replicas from your application and data in the US and Europe
    data centers. As you might guess, this is not as easy as it sounds
    and there are a lot of challenges you need to address, how you can
    effectively redirect the users to the right region, or how you can
    synchronize your data across regions are typical questions or
    concerns you have to tackle first.
-   Availability: you will want to have your applications available all
    the time. Many things can happen, but you need to understand that a
    region or data center might become offline and your application
    should be prepared to handle that scenario. Your applications and
    data should be replicated across multiple regions to not be affected
    when things like this occur.   

Microsoft already offers a set of tools or technologies you can use to
address these two aspects, so let’s discuss some of them in detail in
the next paragraphs.

Windows Azure Traffic Manager
-----------------------------

While this technology is still a CTP and not available for all public in
general, it will provide several methods for distributing internet
traffic among two or more hosted services in different Windows Azure
datacenters. It is in essence a distributed DNS service that knows which
Windows Azure Services are sitting behind the traffic manager URL and
distributes requests based on different policies  or modes you can
configure. It will initially support three modes, “Failover”,
“Performance” and “Round-Robin”.

In the “Failover” mode,  all the traffic is redirected to a single
hosted service, unless it fails. If the redirection fails because the
hosted service is not longer available,  it then directs the traffic to
the hosted service configured as failover. For example, you can
configure the South Central US region as a backup for the North Central
US region and vice versa. If any of those regions go offline, the other
one will take its place.

In the “Performance” mode, all traffic is mapped to the hosted service
“closest” to the client requesting it.  For example, this will direct
users from the US to one of the US datacenters and European users will
probably end up in one of the European datacenters.

Finally, in the “Round-robin” mode, the traffic is distributed in a
round robin fashion across several hosted services configured in the
policy.

As you can see, this tool only tackles the two aspects mentioned before
from the point of view of traffic redirection. However, you also need to
make sure the data is also consistently available in all the regions,
and that’s something this tool won’t solve. For example, if the users
are redirected from North US to Central US, you need to make sure the
data they get access look the same.

Content Delivery Network (CDN)
------------------------------

You can imagine the Windows Azure Content Delivery Network as a huge
http cache spread across the globe for content such as images, files,
scripts, or static html to name a few. You can either configure CDN for
a blob storage account or an http endpoint in your application (and
endpoint with an url ending in “cdn”, for example xxx.cloudapp.net/cdn).
All that content will be cached in the nodes that are part of the
network using standard Http caching. The CDN will serve later the cached
copy closest to the user requesting it, improving in this way the
response time.

There are today more than 18 locations or nodes globally (United States,
Europe, Asia, Australia and South America) and this number keeps
growing.

SQL Azure Data Sync
-------------------

While the Azure Table Service already provides some built-in support for
crash recovery based on the idea of partitions that are replicated
across different nodes (in different sub regions, for example North US
and Central US), a SQL Azure database is not prepared for that
scenario.  If you also use something like the Traffic Manager to
redirect users to the closest data center, you probably would want to
have all the databases in the different regions in sync between them
making all this process transparent to the user.

If you want to support that scenario, the SQL Azure Data Sync is the
tool that will help you in this matter. In a nutshell, SQL Azure Data
Sync is a cloud-based data synchronization service built on the
Microsoft Sync Framework technologies. It provides bi-directional data
synchronization and data management capabilities allowing data to be
easily shared across SQL Azure databases within multiple data centers.

There are a few things you will have to configure in this tool

-   The databases you want to keep in sync, and also the tables and
    columns in those databases
-   The schedule or frequency for doing the synchronization.
-   How you want to resolve any conflict that might occur during the
    synchronization.

As you can see, the combination of this tool with the traffic manager
will help you to keep applications and data in sync between different
region for the most common scenarios.

