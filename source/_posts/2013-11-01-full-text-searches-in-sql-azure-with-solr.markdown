---
layout: post
title: "Full-Text Searches in SQL Azure with Solr"
date: 2013-11-01 15:41
comments: true
categories: azure
---

Solr is a robust search platform created by the open source community on top of Apache Lucene. It's completelly written in java, and uses the Lucene java implementation at is core for full-text indexing and search. In addition, it exposes an http web interface for doing the full text searches and perform management tasks. 
On other hand, we have SQL Azure, which currently does not support full text searches, so these two services complement very well each other. 

As Solr is mainly a java implementation, you only have a few alternatives to run in on Windows Azure. You can deploy it as a worker role together with the java runtime machine, or you can deploy it in a VM. As any solution in the cloud, the state persisted in the worker role or VM goes away when the VMs are replaced or they go down. As Solr persists the indexes in disk, you need to make sure it is stored in a permament storage like Azure Drive or the storage service. If you decide to use a worker role, this requires some additional work to make Solr to store the indexes in Azure drive for example, which is a VHD stored in the storage service that can be mounted by the VM as a local disk. Good news is that MS Open Tech has already done this for us. They have a created a template that deployes Solr with a fail-over configuration(master-slave) in two worker roles, one role for the master node, and another role for the slave node. The slave node replicates from the master node, so in case you lost one of them, you still have the other node available. . In addition, it configures a web role with an MVC application that acts as a admin dashboard for doing basic management stuff. This solution is hosted in Github as part of this project [Windows-Azure-Solr](https://github.com/MSOpenTech/Windows-Azure-Solr). The Github site also provides instructions to get the solution deployed in Windows Azure. 

The template that you download from GitHub imports data into Solr by crawling some URLs. That's part of the data-config.xml file that you can find in the configuration folder of the master and slave nodes (SolrMasterWorkerRole\SolrFiles\data-config.xml and SolrSlaveWorkerRole\SolrFiles\data-config.xml). Solr supports the idea of data importers, which can be used to import data from different sources such as existing web sites, files in disk or even a database.

In this case, we will modify that data-config.xml file to use a data importer that pulls data from an existing SQL Azure instance. The following example shows how this data importer configuration looks like,

```xml
<dataConfig>
  <dataSource type="JdbcDataSource" name="ds1"
    driver="com.microsoft.sqlserver.jdbc.SQLServerDriver" 
    url="jdbc:sqlserver://[your server];database=[your database];user=[your user];password=[your user];encrypt=true;hostNameInCertificate=*.database.windows.net;loginTimeout=30;"
    readOnly="true"
  />

  <document name="articles">
    <entity name="article" dataSource="ds1" pk="id" 
	query = "SELECT id, title, description, tags, author, lastupdated from Articles" 
	deltaQuery="select id FROM Articles WHERE LastUpdated &gt; '${dataimporter.last_index_time}'" 
	deltaImportQuery="SELECT id, title, description, tags, author, lastupdated from Articles where id = '${dataimporter.delta.id}'"
    	>
      <field column="id" name="id" />
      <field column="title" name="title" />
      <field column="description" name="description" />
      <field column="tags" name="tags" />
      <field column="author" name="author" />
      <field column="lastupdated" name="lastupdated" />
    </entity>      
  </document>
</dataConfig>
```
First of all, we have defined a dataSource element that points to our SQL Azure instance. This is a Jdbc data source that uses the SQL Server driver and sets the connection string in the url attribute. 
Secondly, we have defined a document, which specifies one or more entities that are mapped from the SQL Azure database with a select statement. The fields section maps the different fields in the document to the fields returned by the select statements. This document is what Solr stores in the index using Lucene. As you can see, three queries have been defined. The first one with the "query" attribute is used by the data importer when a full import of the complete database is done into Solr. The other two queries are used for supporting a delta or partial import scenario. These two are optional and useful only in scenarios where you have frecuent updates and a lot of data to import, which will make the full import considerably slow.

Since this data importer uses the SQL Server driver for the Jdbc data source, you will have to download that package from the Microsoft website and copying it in the folder where Solr looks for the external libraries (SolrMasterWorkerRole\Solr\dist and SolrSlaveWorkerRole\Solr\dist).

We have defined so far the mapping of a document against one or more tables in the database, but Solr still requires the definition of those fields, which are part of the schema. The schema definition can be found in the schema.xml file (SolrMasterWorkerRole\SolrFiles\v44\schema.xml and SolrSlaveWorkerRole\SolrFiles\v44\schema.xml). The following example shows how the schema is modified to include the fields used by the data importer.

```xml
<field name="id" type="string" indexed="true" stored="true" required="true" multiValued="false" /> 
<field name="title" type="string" indexed="true" stored="true" required="true" multiValued="false" /> 
<field name="description" type="string" indexed="true" stored="true" required="true" multiValued="false" /> 
<field name="tags" type="string" indexed="true" stored="true" required="true" multiValued="false" /> 
<field name="author" type="string" indexed="true" stored="true" required="true" multiValued="false" /> 
<field name="lastupdated" type="date" indexed="true" stored="true" required="true" multiValued="false" />  
```

This is enough to get Solr configured to pull data from SQL Azure, so we are in conditions to deploy Solr into an Windows Azure subscription also using the tool provided by MS Open Tech. If you have installed all the pre-requisites and followed the instructions in the wiki page of the Github website, the following command should be enough to perform that deployment.

Inst4WA.exe -XmlConfigPath "SolrInstWR_V4.4.xml" -subscription "[subscription name]" -location "[location]" -DomainName "[cloud service name]"

If you are lucky enough to get the command to work in the first instance, it will open a new browser instance when the deployment is complete, and it will also redirect you to the MVC dashboard running in the web role. 

You are in conditions now to execute a few commands to import the data into Solr. You will see the location and port where the Solr master and slave nodes are running as part of the home page in the dashboard. For doing a full import into the master role, you have to copy that location and port, and append "/dataimport?command=full-import". For example, "http://mysamplesolr.cloudapp.net:21000/solr/dataimport?command=full-import". That will start a full import that runs asynchronously, so you can use this other command to check the status of the import "/dataimport?command=status". For doing a partial import, you only change the command to this one "/dataimport?command=delta-import". Once the import is complete, you can do a search to verify that everything looks ok. That can be done with the following command "/select?q=[query]". For example, "http://mysamplesolr.cloudapp.net:21000/solr/select?q=azure" 

So you have Solr indexing all your data now, but what happens with the security ?. This thing is open to the world. Anyone can do anything with your Solr instances as everything is public. There are some ways to secure Solr pages by changing some settings in the web server, which is Jetty by default. However, the Solr documentation recommends to put Solr behind a reverse proxy that filters the requests. There are several reverse proxy implementations for Solr in Github, but I will use a different approach for Windows Azure here. Given that the MS Open Tech template includes a web role for the MVC dashboard, and two worker roles for running the Solr master and slave instances, we can make the Solr instances available in internal endpoints only and use the MVC application as a facade or reverse proxy to forward all the requests to these instances. The only public and visible face will be the MVC application in the web role. All the requests for Solr must go through this MVC application first, which can filter any request that looks malicious or any request that can damage the existing indexes. A simple way to do this is to allow only get operations and filter the rest. This is the approach I've been taken and implemented as part of a fork created from the MS OpenTech project in Github. This fork is available [here](https://github.com/pcibraro/Windows-Azure-Solr).      









