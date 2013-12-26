---
layout: post
title: "Monitoring your WCF services with AppFabric"
date: 2010-02-01
comments: true
categories: .NET
---

Windows AppFabric (Previously known as Dublin) introduces a new built-in
mechanism for monitoring the execution of the WCF services, all the
events generated during the execution or the interaction with other
services. While the message trace capabilities in WCF already provided
some detail about the execution and the exchanged messages, it was a
feature more suitable for troubleshooting and error diagnosis.   The
performance counters did not help in this area either, as they provided
live service usage statistics, which sometimes resulted very useful to
determine possible bottlenecks.

When this monitoring mechanism is enabled for IIS Hosted services, the
AppFabric extension will capture different events during the execution
of the services. The level of detail or number of events that you want
to capture is something that can be adjusted through configuration.

In order to configure monitoring for an existing WCF service, you need
to add the following section to the web.config file,

\<microsoft.applicationServer\> \
    \<monitoring\> \
      \
      \<default enabled="true"
connectionStringName="DefaultApplicationServerExtensions"
monitoringLevel="Troubleshooting" /\> \
    \</monitoring\> \
\</microsoft.applicationServer\>

The connectionStringName attribute should contain a reference to an
existing connection string in the ConnectionString section, and that’s
the database used by AppFabric to store the monitoring events.

The monitoringLevel attribute specifies the number of events you want to
capture in a service execution. The possible values for this setting
are,

-   Off (monitoringLevel="Off”): No data is collected. It would be
    equivalent to have monitoring disabled.
-   Errors only (monitoringLevel="ErrorsOnly”):  Only errors and
    warnings events are collected. This is very useful when you don’t
    want to incur into performance issues for collecting application
    events but you still want to know about any error may happen during
    the execution.
-   Health Monitoring (monitoringLevel=HealthMonitoring): This is the
    default level and collects enough application events to show
    different metrics in the AppFabric dashboard.
-   End To End Monitoring (monitoringLevel="EndToEndMonitoring”). It
    collects all the events from the Health Monitoring level, plus
    additional events to reconstruct the message flow. For example,
    Service A called Service B. You will get also events representing
    info about the Service B call if monitoring is enabled on Service A.
-   Troubleshooting (monitoringLevel="Troubleshooting"). This is the
    most verbose of all, so it is appropriate for troubleshooting an
    unhealthy application.

All this monitoring infrastructure is layered on top of “Event Tracing
for Windows (ETW)”, which provides a very efficient and optimized
mechanism for capturing application events without affecting the
application performance at all. In addition, this infrastructure is
complemented with a windows service for collecting the events (A ETW
listener basically), and providers for dumping all the information to
different storages. This last part is totally extensible, and you can
configure your own implementation by implementing the interface
Microsoft.ApplicationServer.Monitoring.EventCollector.IBulkCopy in the
Monitoring.ApplicationServer.Monitoring assembly. This interface looks
quite simple to implement, and AppFabric ships with a default
implementation for SQL Server.

public interface IBulkCopy \
{ \
        int BatchSize { get; set; } \
        DbConnection Connection { get; set; } \
        string DestinationTableName { get; set; }

        void WriteToServer(DataTable dataTable); \
}

The windows service for collecting the ETW events can be found under the
folder,
C:\\Windows\\System32\\ApplicationServerExtensions\\Microsoft.ApplicationServer.ServiceHost.exe,
and it must be started in order to capture and dump the events in the
configured IBulkCopy provider.

In addition, if you want to correlate all the service calls within a
single execution (For example, Service A calling Service B, and B
calling C) under a unique global identifier, so you can group all the
events under that identifier, there is a new \<endToEndTracing\> element
under the \<system.ServiceModel/diagnostics\> section that you can use.

\<system.ServiceModel\> \
  \<diagnostics etwProviderId="7d98f890-ff8d-4311-be4f-e679103ba7cf"\> \
    \<endToEndTracing propagateActivity="true" messageFlowTracing="true"
/\> \
  \</diagnostics\> \
\</system.ServiceModel\>

For example, the following image shows two events that are correlated
under the same E2EActivityId, which represents a service “HelloWorld”
calling an operation in another “EchoService” service.

[![EndToEndTracing](http://weblogs.asp.net/blogs/cibrax/EndToEndTracing_thumb_2D8DD60A.jpg "EndToEndTracing")](http://weblogs.asp.net/blogs/cibrax/EndToEndTracing_51E2E059.jpg)

