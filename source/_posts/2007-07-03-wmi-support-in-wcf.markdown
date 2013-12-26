---
layout: post
title: "WMI Support in WCF"
date: 2007-07-03
comments: true
categories: WCF
---

A nice thing about the management layer in WCF is the WMI support to
expose instrumentation at runtime with a well-know interface. That
instrumentation includes different service characteristics such as
contracts, services, behaviors, listeners or application hosts, which
can be accessed by different management tools.

The built-in provider can be activated in the configuration file of the
application through the "wmiProviderEnabled" attribute in the
diagnostics element, as it is shown below,

\<system.serviceModel\>

    \<diagnostics **wmiProviderEnabled="true"**\>

 

      \<messageLogging logEntireMessage="false"
logMalformedMessages="false" logMessagesAtTransportLevel="false"
logMessagesAtServiceLevel="false"/\>

    \</diagnostics\>

    ...........

\</system.serviceModel\>

Of course, this can be also be done through the SvcConfigEditor tool,
which is the way I usually use for simplicity.

Once the WMI provider is enabled, and the service host is running, we
will able to use a tool like [WMI Object
Browser](http://www.microsoft.com/downloads/details.aspx?familyid=6430F853-1120-48DB-8CC5-F2ABDC3ED314&displaylang=en) to
see the published information.

![](/images/legacy/WMI%20Object%20Browser.jpg)

As is shown in the image above, if we browse to the objects in the
namespace "root\\ServiceModel", we will able to see an AppDomainInfo
object with information about the WCF settings, and the service
endpoints as well.

Also we can change some of those settings at runtime (These changes will
not be saved in the configuration), for instance, to enable message
logging, which can be ideal to detect sporadic problems on a specific
server.

Other settings like Performance counters can also be enabled through
this WMI model, but it requires some programming since it can not easily
done with tools like this one.

And finally, Powershell scripts can also be used to query information
about the objects published by the built-in WMI provider.

PS C:\\Users\\pci\> get-wmiobject -class "AppDomainInfo" -namespace
"root\\servicemodel" -computername "."

\_\_GENUS : 2\
\_\_CLASS : AppDomainInfo\
\_\_SUPERCLASS :\
\_\_DYNASTY : AppDomainInfo\
\_\_RELPATH :
AppDomainInfo.AppDomainId=2,Name="b576ed9b-1-128279485591533267",ProcessId=2184\
\_\_PROPERTY\_COUNT : 12\
\_\_DERIVATION : {}\
\_\_SERVER : PCI-MOBILE\
\_\_NAMESPACE : root\\servicemodel\
\_\_PATH :
\\\\PCI-MOBILE\\root\\servicemodel:AppDomainInfo.AppDomainId=2,Name="b576ed9b-1-128279485591\
533267",ProcessId=2184\
AppDomainId : 2\
IsDefault : False\
LogMalformedMessages : False\
LogMessagesAtServiceLevel : False\
LogMessagesAtTransportLevel : False\
MessageLoggingTraceListeners :\
Name : b576ed9b-1-128279485591533267\
PerformanceCounters : Off\
ProcessId : 2184\
ServiceConfigPath : C:\\Projects\\Service WMI\\service\\web.config\
ServiceModelTraceListeners : {xml}\
TraceLevel : Information

 

