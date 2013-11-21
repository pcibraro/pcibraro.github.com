---
layout: post
title: "Injecting dynamic content in Windows Azure packages"
date: 2013-11-21 10:52
comments: true
categories: azure
---

The Windows Azure Tools 1.7 introduced a new feature for adding content to the Windows Azure projects called "Role Content Folders". In some scenarios, you might want to add custom content such as static pages, documentation, configuration files or external binaries for example. This is useful for example if you want to deploy a solution not written in .NET such as java implementation, and you don't want to mix that with the .NET code in the Role project. The following image shows how the content folders are added in the Azure project.

![Content Folders](/images/azure_content/content_folders.png "Content Folders")

That content is included in the generated Azure package, and it is deployed in the AppRoot folder when the package is finally published in VM in the cloud. 

One of the problem with this feature is that you might want to include content with an structure that changes often or content with thousands of folders/files, which requires some tedious manual work in Visual Studio to keep that content updated in the project.

Good news is that you can use a MSBuild task to inject that custom content automatically when the package is being generated. You have to include a custom Target "BeforeRoleAddContent" right after the declaration of the "Microsoft.WindowsAzure.targets" as it is shown bellow,

```xml
<Import Project="$(CloudExtensionsDir)Microsoft.WindowsAzure.targets" />

<Target Name="BeforeAddRoleContent">
	<ItemGroup>
	  <AzureRoleContent Include="..\Solr\Solr">
	    <RoleName>SolrMasterHostWorkerRole</RoleName>
	    <Destination>Solr</Destination>
	  </AzureRoleContent>
	  <AzureRoleContent Include="..\Solr\jre6">
	    <RoleName>SolrMasterHostWorkerRole</RoleName>
	    <Destination>jre6</Destination>
	  </AzureRoleContent>
	  <AzureRoleContent Include="..\Solr\SolrFiles">
	    <RoleName>SolrMasterHostWorkerRole</RoleName>
	    <Destination>SolrFiles</Destination>
	  </AzureRoleContent>
	  <AzureRoleContent Include="..\Solr\Solr">
	    <RoleName>SolrSlaveHostWorkerRole</RoleName>
	    <Destination>Solr</Destination>
	  </AzureRoleContent>
	  <AzureRoleContent Include="..\Solr\jre6">
	    <RoleName>SolrSlaveHostWorkerRole</RoleName>
	    <Destination>jre6</Destination>
	  </AzureRoleContent>
	  <AzureRoleContent Include="..\Solr\SolrFiles">
	    <RoleName>SolrSlaveHostWorkerRole</RoleName>
	    <Destination>SolrFiles</Destination>
	  </AzureRoleContent>
	</ItemGroup>
</Target>
```

The example above injects several content folders in two different roles, "SolrMasterHostWorkerRole" and "SolrSlaveHostWorkerRole". The "Include" attribute specifies the source folder, and the Destination folder within the AppRoot is specified in the Destination element. 







