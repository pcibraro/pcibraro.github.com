---
layout: post
title: "Monitoring your WCF Web Apis with AppFabric"
date: 2011-03-07
comments: true
categories: .NET
---

The other day, [Ron Jacobs](http://blogs.msdn.com/b/rjacobs/) made
public a
[template](http://blogs.msdn.com/b/rjacobs/archive/2011/03/03/appfabric-wcf-http-service-template.aspx)
in the Visual Studio Gallery for enabling monitoring capabilities to any
existing WCF Http service hosted in Windows AppFabric. I thought it
would be a cool idea to reuse some of that for doing the same thing on
the new WCF Web Http stack.

Windows AppFabric provides a dashboard that you can use to dig into some
metrics about the services usage, such as number of calls, errors or
information about different events during a service call. Those events
not only include information about the WCF pipeline, but also custom
events that any developer can inject and make sense for troubleshooting
issues.

[![AppFabric\_Dashboard](http://weblogs.asp.net/blogs/cibrax/AppFabric_Dashboard_thumb_78F79A10.png "AppFabric_Dashboard")](http://weblogs.asp.net/blogs/cibrax/AppFabric_Dashboard_6A106AF6.png)    

This monitoring capabilities can be enabled on any specific IIS virtual
directory by using the AppFabric configuration tool or adding the
following configuration sections to your existing web app,

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: white; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
<system.serviceModel>
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
    <serviceHostingEnvironment aspNetCompatibilityEnabled="true" multipleSiteBindingsEnabled="true" />
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: white; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
    <diagnostics etwProviderId="3e99c707-3503-4f33-a62d-2289dfa40d41">
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
        <endToEndTracing propagateActivity="true" messageFlowTracing="true" />
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: white; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
    </diagnostics>
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
    <behaviors>
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: white; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
        <serviceBehaviors>
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
            <behavior name="">
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: white; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
                <etwTracking profileName="EndToEndMonitoring Tracking Profile" />
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
            </behavior>
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: white; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
        </serviceBehaviors>
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
    </behaviors>
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: white; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
</system.serviceModel>
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
 
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: white; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
<microsoft.applicationServer>
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
    <monitoring>
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: white; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
        <default enabled="true" connectionStringName="ApplicationServerMonitoringConnectionString" monitoringLevel="EndToEndMonitoring" />
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
    </monitoring>
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: white; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
</microsoft.applicationServer>
~~~~

Bad news is that none of the configuration above can be easily set on
code by using the new configuration model for WCF Web stack.  A good
thing is that you easily disable it in the configuration when you no
longer need it, and also uses ETW, a general-purpose and high-speed
tracing facility provided by the operating system (it’s part of the
windows kernel).

By adding that configuration section, AppFabric will start monitoring
your service automatically and providing some basic event information
about the service calls. You need some custom code for injecting custom
events in the monitoring data.

What I did here is to copy and refactor the “WCFUserEventProvider” class
provided as sample in the Ron’s template to make it more TDD friendly
when using IoC. I created a simple interface “ILogger” that any service
(or resource) can use to inject custom events or monitoring information
in the AppFabric database.

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: white; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
public interface ILogger
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
{
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: white; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
    bool WriteError(string name, string format, params object[] args);
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
    bool WriteWarning(string name, string format, params object[] args);
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: white; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
    bool WriteInformation(string name, string format, params object[] args);
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
}
~~~~

The “WCFUserEventProvider” class implements this interface by making
possible to send the events to the AppFabric monitoring database. The
service or resource implementation can receive an “ILogger” as part of
the constructor.

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: white; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
[ServiceContract]
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
[Export]
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: white; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
public class OrderResource
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
{
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: white; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
    IOrderRepository repository;
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
    ILogger logger;
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: white; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
 
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
    [ImportingConstructor]
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: white; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
    public OrderResource(IOrderRepository repository, ILogger logger)
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
    {
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: white; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
        this.repository = repository;
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
        this.logger = logger;
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: white; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
    }
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
 
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: white; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
    [WebGet(UriTemplate = "{id}")]
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
    public Order Get(string id, HttpResponseMessage response)
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: white; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
    {
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
        var order = this.repository.All.FirstOrDefault(o => o.OrderId == int.Parse(id, CultureInfo.InvariantCulture));
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: white; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
        if (order == null)
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
        {
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: white; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
            response.StatusCode = HttpStatusCode.NotFound;
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
            response.Content = new StringContent("Order not found");
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: white; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
        }
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
 
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: white; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
        this.logger.WriteInformation("Order Requested", "Order Id {0}", id); 
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
 
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: white; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
        return order;
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
    }
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: white; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
}
~~~~

The example above uses “MEF” as IoC for injecting a repository and the
logger implementation into the service. You can also see how the logger
is used to write an information event in the monitoring database.

The following image illustrates how the custom event is injected and the
information becomes available for any user in the dashboard.

[![AppFabric\_Details](http://weblogs.asp.net/blogs/cibrax/AppFabric_Details_thumb_2E603669.png "AppFabric_Details")](http://weblogs.asp.net/blogs/cibrax/AppFabric_Details_36A481CD.png)

An issue that you might run into and I hope the WCF and AppFabric teams
fixed soon is that any WCF service that uses friendly URLs with ASP.NET
routing does not get listed as a available service in the WCF services
tab in the AppFabric console.

The complete example is available to download from
[here](http://kioskotobys.com.ar/WebApisMonitoring.zip).

