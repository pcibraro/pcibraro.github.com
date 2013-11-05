---
layout: post
title: "Full-Text Searches in SQL Azure with Solr"
date: 2013-11-01 15:41
comments: true
categories: azure
---

Solr is a robust search platform created by the open source community on top of Apache Lucene. It's completelly written in java, and uses the Lucene java implementation at is core for full-text indexing and search. In addition, it exposes an http web interface for doing the full text searches and perform management tasks. 
On other hand, we have SQL Azure, which currently does not support full text searches, so these two services complement very well each other. 

As Solr is mainly a java implementation, you only have a few alternatives to run in on Windows Azure. You can deploy it as a worker role together with the java runtime machine, or you can deploy it in a VM. As any solution in the cloud, the state persisted in the worker role or VM goes away when the VMs are replaced or they go down. As Solr persists the indexes in disk, you need to make sure it is stored in a permament storage like Azure Drive or the storage service. If you decide to use a worker role, this requires some additional work to make Solr to store the indexes in Azure drive for example. Good news is that MS Open Tech has already done this for us. They have a created a template that deployes Solr with a fail-over configuration(master-slave) in two worker roles, one role for the master node, and another role for the slave node. The slave node replicates from the master node, so in case you lost one of them, you still have the other node available. In addition, it configures a web role with an MVC application that acts as a admin dashboard for doing basic management stuff. This solution is hosted in Github as part of this project [Windows-Azure-Solr](https://github.com/MSOpenTech/Windows-Azure-Solr). The Github site also provides instructions to get the solution deployed in Windows Azure. 

The template that you download from GitHub imports data into Solr by crawling some URLs. That's part of the data-config.xml file that you can find in the configuration folder of the master and slave nodes. Solr supports the idea of data importers, which can be used to import data from different sources such as existing web sites, files in disk or even a database. 




