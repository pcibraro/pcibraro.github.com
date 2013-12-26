---
layout: post
title: "Client Configuration in WCF 4.0"
date: 2009-09-08
comments: true
categories: .NET
---

As Dr Nick announced in this
[post](http://blogs.msdn.com/drnick/archive/2009/08/24/what-s-new-in-wcf-4-more-on-services-and-configuration.aspx),
WCF 4.0 will ship with a new feature to configure a client channel from
a configuration source other than the traditional section in the
application configuration file (I discussed a workaround for doing the
same thing in WCF 3.0 a time ago in this
[post](http://weblogs.asp.net/cibrax/archive/2007/10/19/loading-the-wcf-configuration-from-different-files-on-the-client-side.aspx))

This will provide the flexibility to deploy some binaries with the
client proxies together with an independent configuration file (which
might be a resource file, a stand alone file included with your
application or something you can download from an specific place), so
the main configuration file does not need to be touched at all.

A new channel factory “ConfigurationChannelFactory” is included as part
of the System.ServiceModel.Configuration namespace to create channels
from an alternative configuration source.

**var fileMap = new ExeConfigurationFileMap(); \
            fileMap.ExeConfigFilename = "otherFile.config";**

**var configuration =
ConfigurationManager.OpenMappedExeConfiguration(fileMap,
ConfigurationUserLevel.None);**

**var factory = new
ConfigurationChannelFactory\<IService1\>("BasicHttpBinding\_IService1",
configuration, new
EndpointAddress("**[**http://localhost:49187/Service1.svc**](http://localhost:49187/Service1.svc)**"));**

var client = factory.CreateChannel();

var s = client.GetData(3);

Console.WriteLine(s);

The file “otherFile.config” is just a simple configuration file that
contains the client definition and configuration for using the
“IService1” service.

\<configuration\> \
  \
    \<system.serviceModel\> \
        \<bindings\> \
            \<basicHttpBinding\> \
              \<binding name="BasicHttpBinding\_IService1" \> \
                    \<security mode="None"\> \
                    \</security\> \
                \</binding\> \
            \</basicHttpBinding\> \
        \</bindings\> \
        \<client\> \
            \<endpoint
address="[http://localhost:49187/Service1.svc](http://localhost:49187/Service1.svc)"
binding="basicHttpBinding" \
                bindingConfiguration="BasicHttpBinding\_IService1"
contract="ServiceReference1.IService1" \
                name="BasicHttpBinding\_IService1" /\> \
        \</client\> \
    \</system.serviceModel\> \
\</configuration\>

All this code is based on the .NET 4.0 beta 1, so it might change in the
final release.

