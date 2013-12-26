---
layout: post
title: "Generating WCF configuration from the SO-Aware Repository"
date: 2010-11-10
comments: true
categories: SO-Aware
---

As part of the simplification in service configuration that we want to
provide in SO-Aware, we have added two new commands in the PowerShell
provider for generating the service configuration at design time in case
you don’t want to rely on SO-Aware for resolving all that at runtime.

We have added a command for generating the configuration specific for
SO-Aware

**Get-SWServiceConfiguration** -serviceversion "SampleService(1.0)"
-configpath "d:\\temp\\MyConfig.config" -servicetype
"Services.SampleService"

The arguments you pass to this command are the service version (A
service that you already imported in the repository), the path to the
configuration file, and the .NET type implementing the service. As
result of executing this command, the following section will be added to
the configuration file.

\<configuration\> \
  \<configSections\> \
    \<section name="serviceRepository"
type="Tellago.ServiceModel.Governance.ServiceConfiguration.ServiceRepositoryConfigurationSection,
Tellago.ServiceModel.Governance.ServiceConfiguration" /\> \
  \</configSections\> \
  \<serviceRepository
url="[http://localhost/soaware/servicerepository.svc](http://localhost/soaware/servicerepository.svc)"\>
\
    \<services\> \
      \<service name="ref:SampleService(1.0)"
type="Services.SampleService" /\> \
    \</services\> \
  \</serviceRepository\>

\</configuration\>

That’s the configuration required by SO-Aware to resolve the service
configuration at runtime.

In addition, if you don’t want to rely on SO-Aware at all, another
command has been added to generate the WCF configuration section for a
service. This command might also becomes handy if you want to deploy the
configuration at design time in a cluster for example.

**Get-SWWCFServiceConfiguration** -serviceversion "SampleService(1.0)"
-configpath "d:\\temp\\MyConfig.config" -servicetype
"Samples.SampleService"

\<configuration\> \
  \<system.serviceModel\> \
    \<services\>     \
      \<service name="Samples.SampleService"\> \
        \<endpoint
address="[http://localhost:13749/SampleService.svc](http://localhost:13749/SampleService.svc)"
contract="ISampleService" binding="basicHttpBinding"
bindingConfiguration="basicHttp" /\> \
      \</service\> \
    \</services\> \
    \<behaviors\> \
      \<serviceBehaviors /\> \
      \<endpointBehaviors /\> \
    \</behaviors\> \
    \<bindings\> \
      \<basicHttpBinding\> \
        \<binding name="basicHttp" /\> \
      \</basicHttpBinding\> \
    \</bindings\> \
  \</system.serviceModel\> \
\</configuration\>

Both commands receive the same arguments, but the generated
configuration is quite different as you can see.

