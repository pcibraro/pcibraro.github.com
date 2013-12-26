---
layout: post
title: "ConfigSource attribute on system.serviceModel section"
date: 2007-07-24
comments: true
categories: .NET
---

The configSource attribute was firstly introduced in .NET framework 2.0
to support external configuration files.

This attribute can be added to any configuration section to specify a an
external file for that section. Using an external configuration source
can be useful in many scenarios. For instance, you could place a section
into an external configSource if you need an easy method to swap
settings for the section depending on the environment (development,
test, or production), or  you need granular control over permissions.

Unfortunately, the system.serviceModel section group does not support
this attribute. If you try to add it, you will receive the following
exception:

***The attribute 'configSource' cannot be specified because its name
starts with the reserved prefix 'config' or 'lock'***

What I found out is that you can use this attribute on the different
sections under system.serviceModel such as services, behaviors or
bindings.

For instance, the configuration file could look like this,

\<configuration\>

 

  \<system.serviceModel\>

    \<services configSource="Services.config" \>

    \</services\>

 

    \<bindings configSource="Bindings.config"\>

    \</bindings\>

 

    \<behaviors configSource="Behaviors.config"\>

    \</behaviors\>

 

  \</system.serviceModel\>

 

\</configuration\>

 And then, each file contains the corresponding section.

**Services.config**

\<services\>

  \<service name="Microsoft.ServiceModel.Samples.CalculatorService"

          behaviorConfiguration="CalculatorServiceBehavior"\>

    \<host\>

      \<baseAddresses\>

        \<add
baseAddress="http://localhost:8000/servicemodelsamples/service"/\>

      \</baseAddresses\>

    \</host\>

    \<!-- this endpoint is exposed at:
net.tcp://localhost:9000/servicemodelsamples/service  --\>

    \<endpoint
address="net.tcp://localhost:9000/servicemodelsamples/service"

              binding="netTcpBinding"

              bindingConfiguration="Binding1"

              contract="Microsoft.ServiceModel.Samples.ICalculator" /\>

    \<!-- the mex endpoint is exposed at
http://localhost:8000/ServiceModelSamples/service/mex --\>

    \<endpoint address="mex"

              binding="mexHttpBinding"

              contract="IMetadataExchange" /\>

  \</service\>

\</services\>

**Bindings.config**

\<bindings\>

  \<netTcpBinding\>

    \<binding name="Binding1"

            closeTimeout="00:01:00"

            openTimeout="00:01:00"

            receiveTimeout="00:10:00"

            sendTimeout="00:01:00"

            transactionFlow="false"

            transferMode="Buffered"

            transactionProtocol="OleTransactions"

            hostNameComparisonMode="StrongWildcard"

            listenBacklog="10"

            maxBufferPoolSize="524288"

            maxBufferSize="65536"

            maxConnections="10"

            maxReceivedMessageSize="65536"\>

      \<readerQuotas maxDepth="32"

                    maxStringContentLength="8192"

                    maxArrayLength="16384"

                    maxBytesPerRead="4096"

                    maxNameTableCharCount="16384" /\>

      \<reliableSession ordered="true"

                      inactivityTimeout="00:10:00"

                      enabled="false" /\>

      \<security mode="Transport"\>

        \<transport clientCredentialType="Windows"
protectionLevel="EncryptAndSign" /\>

      \</security\>

    \</binding\>

  \</netTcpBinding\>

\</bindings\>

**Behaviors.config**

\<behaviors\>

  \<serviceBehaviors\>

    \<behavior name="CalculatorServiceBehavior"\>

      \<serviceMetadata httpGetEnabled="true" /\>

      \<serviceDebug includeExceptionDetailInFaults="False" /\>

    \</behavior\>

  \</serviceBehaviors\>

\</behaviors\>

