---
layout: post
title: "Why ASMX web services are not an excuse anymore with WCF 4.0"
date: 2010-09-06
comments: true
categories: .NET
---

ASXM web services has been the favorite choice for many developers for
building soap web services in .NET during a long time because of its
simplicity. With ASMX web services, you get a web service up and running
in a matter of seconds, as it does not require any configuration. The
only thing you need to do is to build the service implementation and the
message contracts (xml serialization classes), and that’s all. However,
when you build a system as a black box with most of the configuration
hardcoded, and only a few extensibility points in mind, you will
probably end up with something that is very easy to deploy and get
running, but it can not be customized at all. That’s what an ASMX web
service is after all, you don’t have a way easily change the protocol
versions, encoders, security or even extend with custom functionality
(SOAP extensions are the only entry point for extensibility, which work
as message inspectors in WCF).

On the other hand, you have WCF, which is extensible beast for building
services among other things. The number of extensibility points that you
will find in WCF is extremely high, but the downside is that
configuration also becomes extremely complex and a nightmare for most
developers that only want to get their services up and running.

Fortunately, the WCF team has considerably improved the configuration
experience in WCF 4.0, making possible to run a service with almost no
configuration. The approach that they have taken for this version is to
make everything work with no configuration, and give the chance to
override what you actually need for a given scenario.

For instance, a WCF service that uses http as transport behaves a ASMX
web service by default (it uses basicHttpBinding with SOAP 1.2,
transport security, text encoding and Basic profile 1.1) unless you
change that. So, how can you create a new WCF service as you did before
with ASMX ?. That’s simple and you need to follow these steps,

​1. Create a new WCF service in Visual Studio

[![visualstudio\_newservice](http://weblogs.asp.net/blogs/cibrax/visualstudio_newservice_thumb_38629E46.png "visualstudio_newservice")](http://weblogs.asp.net/blogs/cibrax/visualstudio_newservice_47C62622.png)

​2. Modify the service and data contract to expose the operations you
actually need in the service.

 

~~~~ {#codeSnippet style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: #f4f4f4; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
// NOTE: You can use the "Rename" command on the "Refactor" menu to change the interface name "IService1" in both code and config file together.[ServiceContract]public interface IService1{    [OperationContract]    string GetData(int value);    [OperationContract]    CompositeType GetDataUsingDataContract(CompositeType composite);    // TODO: Add your service operations here}// Use a data contract as illustrated in the sample below to add composite types to service operations.[DataContract]public class CompositeType{    bool boolValue = true;    string stringValue = "Hello ";    [DataMember]    public bool BoolValue    {        get { return boolValue; }        set { boolValue = value; }    }    [DataMember]    public string StringValue    {        get { return stringValue; }        set { stringValue = value; }    }}
~~~~

\

​3. Optionally, enable the service metadata page for the service, so any
client application can use this to generate the proxies.

~~~~ {#codeSnippet style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: #f4f4f4; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
<system.serviceModel>    <behaviors>        <serviceBehaviors>            <behavior>                <serviceMetadata httpGetEnabled="true"/>            </behavior>        </serviceBehaviors>    </behaviors></system.serviceModel>
~~~~

\

 

​4. Optionally, enable the ASP.NET Compatibility mode to use the ASP.NET
security context (Otherwise, the service will use the default security
settings for the basicHttpBinding). That will require two additional
steps, adding the “serviceHostingEnvironment” element in the existing
service model configuration.

~~~~ {#codeSnippet style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: #f4f4f4; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
<system.serviceModel>    <serviceHostingEnvironment aspNetCompatibilityEnabled="true"/>
~~~~

\

 

And adding an attribute in the service,

~~~~ {#codeSnippet style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: #f4f4f4; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
[AspNetCompatibilityRequirements(RequirementsMode=AspNetCompatibilityRequirementsMode.Allowed)]public class Service1 : IService1
~~~~

\

That’s all you need to implement a new WCF service that will behave as a
traditional ASMX webservice. As you can see, no service or binding
configurations were required for the service. In addition, the behavior
element does not have any name, so it applies to all the services
running in the same host.

