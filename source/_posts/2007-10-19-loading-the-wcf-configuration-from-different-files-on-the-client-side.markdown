---
layout: post
title: "Loading the WCF configuration from different files on the client side"
date: 2007-10-19
comments: true
categories: WCF
---

A common problem in WCF that many people face is the impossibility of
loading the client configuration from different configuration files.
This is a common scenario  when the developer wants to
deploy some binaries with along with an independent configuration file
(Which may be in a resource file also) to avoid modifying the main
configuration file. A weeks ago, I described a easy workaround to use
the external configuration files through the use of the configSource
attribute (A mechanism provided by .NET). However, that approach
requires several configuration files to configure an entire WCF channel,
at least three files (client section, bindings and behaviors).

Another approach, which is less common and I will describe in this post,
requires some custom code to extend the ChannelFactory\<T\> class.

The ChannelFactory\<T\> provides a virtual method "CreateDescription"
that can be overridden to create a ServiceEndpoint.

 

//

// Summary:

//    Creates a description of the service endpoint.

//

// Returns:

//    The System.ServiceModel.Description.ServiceEndpoint of the
service.

//

// Exceptions:

//   System.InvalidOperatorException:

//    The callback contract is null but the service endpoint requires
one that

//    is non-null.

protected override ServiceEndpoint CreateDescription();

The default implementation of this method basically tries to load
the endpoint configuration from the default configuration file (The file
configured for the default AppDomain). So, the workaround here is to
override that method and load the configuration from another file. It
seems to be pretty easy to do at first glance, but as you will see next,
it requires a lot of plumbing code to make it work.

The first step is to derive our custom channel class from the
ChannelFactory\<T\> class and add an argument in the constructor to
specify a different configuration file.

/// \<summary\>

/// Custom client channel. Allows to specify a different configuration
file

/// \</summary\>

/// \<typeparam name="T"\>\</typeparam\>

public class CustomClientChannel\<T\> : ChannelFactory\<T\>

{

  string configurationPath;

 

  /// \<summary\>

  /// Constructor

  /// \</summary\>

  /// \<param name="configurationPath"\>\</param\>

  public CustomClientChannel(string configurationPath) : base(typeof(T))

  {

    this.configurationPath = configurationPath;

    base.InitializeEndpoint((string)null, null);

  }

As you can see, a call to the method InitialiazeEndpoint of the base
class is required. That method will automatically call to our
CreateDescription method to configure the service endpoint.

/// \<summary\>

/// Loads the serviceEndpoint description from the specified
configuration file

/// \</summary\>

/// \<returns\>\</returns\>

protected override ServiceEndpoint CreateDescription()

{

   ServiceEndpoint serviceEndpoint = base.CreateDescription();

 

   ExeConfigurationFileMap map = new ExeConfigurationFileMap();

   map.ExeConfigFilename = this.configurationPath;

 

   Configuration config =
ConfigurationManager.OpenMappedExeConfiguration(map,
ConfigurationUserLevel.None);

   ServiceModelSectionGroup group =
ServiceModelSectionGroup.GetSectionGroup(config);

 

   ChannelEndpointElement selectedEndpoint = null;

 

   foreach (ChannelEndpointElement endpoint in group.Client.Endpoints)

   {

      if (endpoint.Contract ==
serviceEndpoint.Contract.ConfigurationName)

      {

        selectedEndpoint = endpoint;

        break;

      }

    }

 

    if (selectedEndpoint != null)

    {

      if (serviceEndpoint.Binding == null)

      {

        serviceEndpoint.Binding =
CreateBinding(selectedEndpoint.Binding, group);

      }

 

      if (serviceEndpoint.Address == null)

      {

        serviceEndpoint.Address = new
EndpointAddress(selectedEndpoint.Address,
GetIdentity(selectedEndpoint.Identity),
selectedEndpoint.Headers.Headers);

      }

 

      if (serviceEndpoint.Behaviors.Count == 0 &&
selectedEndpoint.BehaviorConfiguration != null)

      {

        AddBehaviors(selectedEndpoint.BehaviorConfiguration,
serviceEndpoint, group);

      }

 

      serviceEndpoint.Name = selectedEndpoint.Contract;

    }

 

    return serviceEndpoint;

 

}

 

CreateDescription is the core method where we create the service
endpoint using the configuration file specified in the constructor
method.

And the last step, is to use our custom channel in the client
application instead of the base channel provided by WCF.

CustomClientChannel\<ICalculator\> channel = new
CustomClientChannel\<ICalculator\>("OtherConfig.config");

ICalculator client = channel.CreateChannel();

The code below uses the ICalculator contract  that comes with the WCF
sdk samples.

You can download the [complete sample
here](/images/legacy/CustomChannel.zip)

 

 

 

