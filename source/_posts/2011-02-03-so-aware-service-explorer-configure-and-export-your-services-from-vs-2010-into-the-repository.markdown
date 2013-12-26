---
layout: post
title: "SO-Aware Service Explorer – Configure and Export your services from VS 2010 into the repository"
date: 2011-02-03
comments: true
categories: .NET
---

We have introduced a new Visual Studio tool called “Service Explorer” as
part of the new [SO-Aware SDK version
1.3](http://www.tellagostudios.com/projectfiles/SO-Aware_SDK_1.3.zip) to
help developers to configure and export any regular WCF service into the
SO-Aware service repository.

This new tool is a regular Visual Studio Tool Window that can be opened
from “View –\> Other Windows –\> Services Explorer”.

[![ServicesExplorer\_VS](http://weblogs.asp.net/blogs/cibrax/ServicesExplorer_VS_thumb_65A8C7AD.png "ServicesExplorer_VS")](http://weblogs.asp.net/blogs/cibrax/ServicesExplorer_VS_6976924A.png)

Once you open the Services Explorer, you will able to see all the
available WCF services in the Visual Studio Solution.

[![ServicesExplorer\_VS2](http://weblogs.asp.net/blogs/cibrax/ServicesExplorer_VS2_thumb_2847C74C.png "ServicesExplorer_VS2")](http://weblogs.asp.net/blogs/cibrax/ServicesExplorer_VS2_3883B512.png)

In the image above, you can see that a “HelloWorld” service was found in
the solution and listed under the Tool window on the left. There are two
things you can do for a new service in tool, you can either export it to
SO-Aware repository or associate it to an existing service version in
the repository.

Exporting the service to SO-Aware means that you want to create a new
service version in the repository and associate the WCF service WSDL to
that version. Associating the service means that you want to use a
version already created in SO-Aware with the only purpose of managing
and centralizing the service configuration in SO-Aware.

The option for exporting a service will popup a dialog like the one
bellow in which you can enter some basic information about the service
version you want to create and the repository location.

[![ServicesExplorer\_VS3](http://weblogs.asp.net/blogs/cibrax/ServicesExplorer_VS3_thumb_7C674D8F.png "ServicesExplorer_VS3")](http://weblogs.asp.net/blogs/cibrax/ServicesExplorer_VS3_1E332621.png)

The option for associating a service will popup a dialog in which you
can pick any existing service version repository and the application
configuration file that you want to keep in sync for the service
configuration. Two options are available for configuring a service, WCF
Configuration or SO-Aware. The WCF Configuration option just tells the
tool that the service will use the standard WCF configuration section
“system.serviceModel” but that section must be updated and kept in sync
with the configuration selected for the service in the repository. The
SO-Aware configuration option will tell the tool that the service
configuration will be resolved at runtime from the repository.

[![ServicesExplorer\_VS4](http://weblogs.asp.net/blogs/cibrax/ServicesExplorer_VS4_thumb_4F423AF4.png "ServicesExplorer_VS4")](http://weblogs.asp.net/blogs/cibrax/ServicesExplorer_VS4_1A655B84.png)

For example, selecting SO-Aware will generate the following
configuration in the selected application configuration file,

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: white; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
<configuration>
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
  <configSections>
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: white; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
    <section name="serviceRepository" type="Tellago.ServiceModel.Governance.ServiceConfiguration.ServiceRepositoryConfigurationSection, Tellago.ServiceModel.Governance.ServiceConfiguration" />
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
  </configSections>
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: white; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
  <serviceRepository url="http://localhost/soaware/servicerepository.svc">
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
    <services>
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: white; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
      <service name="ref:HelloWorldService(1.0)@dev" type="SOAwareSampleService.HelloWorldService" />
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
    </services>
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: white; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
  </serviceRepository>
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
</configuration>
~~~~

As you can see the tool represents a great addition to the toolset that
any developer can use to manage and centralize configuration for WCF
services. In addition, it can be combined with other useful tools like
WSCF.Blue (Web Service Contract First) for generating the service
artifacts like schemas, service code or the service WSDL itself.

