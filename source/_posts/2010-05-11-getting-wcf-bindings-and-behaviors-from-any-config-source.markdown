---
layout: post
title: "Getting WCF Bindings and Behaviors from any config source"
date: 2010-05-11
comments: true
categories: .NET
---

The need of loading WCF bindings or behaviors from different sources
such as files in a disk or databases is a common requirement when
dealing with configuration either on the client side or the service
side.

The traditional way to accomplish this in WCF is loading everything from
the standard configuration section (serviceModel section) or creating
all the bindings and behaviors by hand in code. However, there is a
solution in the middle that becomes handy when more flexibility is
needed. This solution involves getting the configuration from any place,
and use that configuration to automatically configure any existing
binding or behavior instance created with code. 

In order to configure a binding instance
(System.ServiceModel.Channels.Binding) that you later inject in any
endpoint on the client channel or the service host, you first need to
get a binding configuration section from any configuration file (you can
generate a temp file on the fly if you are using any other source for
storing the configuration).

 

~~~~ {.code}
private BindingsSection GetBindingsSection(string path)
{
  System.Configuration.Configuration config = 
~~~~

~~~~ {.code}
System.Configuration.ConfigurationManager.OpenMappedExeConfiguration(
    new System.Configuration.ExeConfigurationFileMap() { ExeConfigFilename = path },
      System.Configuration.ConfigurationUserLevel.None);

  var serviceModel = ServiceModelSectionGroup.GetSectionGroup(config);
  return serviceModel.Bindings;
}
~~~~

[](http://11011.net/software/vspaste)

 

The BindingsSection contains a list of all the configured bindings in
the serviceModel configuration section, so you can iterate through all
the configured binding that get the one you need (You don’t need to have
a complete serviceModel section, a section with the bindings only
works).

 

~~~~ {.code}
public Binding ResolveBinding(string name)
{
  BindingsSection section = GetBindingsSection(path);

  foreach (var bindingCollection in section.BindingCollections)
  {
    if (bindingCollection.ConfiguredBindings.Count > 0 
~~~~

~~~~ {.code}
&& bindingCollection.ConfiguredBindings[0].Name == name)
    {
      var bindingElement = bindingCollection.ConfiguredBindings[0];
      var binding = (Binding)Activator.CreateInstance(bindingCollection.BindingType);
      binding.Name = bindingElement.Name;
      bindingElement.ApplyConfiguration(binding);

      return binding;
    }
  }

  return null;
}
~~~~

[](http://11011.net/software/vspaste)

 

The code above does just that, and also instantiates and configures the
Binding object (System.ServiceModel.Channels.Binding) you are looking
for. As you can see, the binding configuration element contains a method
“ApplyConfiguration” that receives the binding instance that needs to be
configured.

A similar thing can be done for instance with the “Endpoint” behaviors.
You first get the BehaviorsSection, and then, the behavior you want to
use.

 

~~~~ {.code}
private BehaviorsSection GetBehaviorsSection(string path)
{
  System.Configuration.Configuration config = 
~~~~

~~~~ {.code}
System.Configuration.ConfigurationManager.OpenMappedExeConfiguration(
       new System.Configuration.ExeConfigurationFileMap() { ExeConfigFilename = path },
          System.Configuration.ConfigurationUserLevel.None);

  var serviceModel = ServiceModelSectionGroup.GetSectionGroup(config);
  return serviceModel.Behaviors;

}
~~~~

~~~~ {.code}
public List<IEndpointBehavior> ResolveEndpointBehavior(string name)
{
  BehaviorsSection section = GetBehaviorsSection(path);
  List<IEndpointBehavior> endpointBehaviors = new List<IEndpointBehavior>();

  if (section.EndpointBehaviors.Count > 0 
~~~~

~~~~ {.code}
&& section.EndpointBehaviors[0].Name == name)
  {
    var behaviorCollectionElement = section.EndpointBehaviors[0];

    foreach (BehaviorExtensionElement behaviorExtension in behaviorCollectionElement)
    {
      object extension = behaviorExtension.GetType().InvokeMember("CreateBehavior",
            BindingFlags.InvokeMethod | BindingFlags.NonPublic | BindingFlags.Instance,
            null, behaviorExtension, null);

      endpointBehaviors.Add((IEndpointBehavior)extension);
    }

   return endpointBehaviors;
 }

 return null;
}
~~~~

[](http://11011.net/software/vspaste)

 

In this case, the code for creating the behavior instance is more
tricky. First of all, a behavior in the configuration section actually
represents a set of “IEndpoint” behaviors, and the behavior element you
get from the configuration does not have any public method to configure
an existing behavior instance. This last one only contains a protected
method “CreateBehavior” that you can use for that purpose.

Once you get this code implemented, a client channel can be easily
configured as follows

 

~~~~ {.code}
var binding = resolver.ResolveBinding("MyBinding");
var behaviors = resolver.ResolveEndpointBehavior("MyBehavior");

SampleServiceClient client = new SampleServiceClient(binding, 
       new EndpointAddress(new Uri("http://localhost:13749/SampleService.svc"),
       new DnsEndpointIdentity("localhost")));
            
foreach (var behavior in behaviors)
{
   if(client.Endpoint.Behaviors.Contains(behavior.GetType()))
   {
     client.Endpoint.Behaviors.Remove(behavior.GetType());
   }
   client.Endpoint.Behaviors.Add(behavior);
}
~~~~

 

The code above assumes that a configuration file (in any place) with a
binding “MyBinding” and a behavior “MyBehavior” exists. That file can
look like this,

 

~~~~ {.code}
<system.serviceModel>
   <bindings>
     <basicHttpBinding>
       <binding name="MyBinding">
         <security mode="Transport"></security>
       </binding>
     </basicHttpBinding>
   </bindings>
   <behaviors>
     <endpointBehaviors>
       <behavior name="MyBehavior">
         <clientCredentials>
           <windows/>
         </clientCredentials>
       </behavior>
     </endpointBehaviors>
   </behaviors>
 </system.serviceModel>
~~~~

[](http://11011.net/software/vspaste)

 

The same thing can be done of course in the service host if you want to
manually configure the bindings and behaviors.

[](http://11011.net/software/vspaste) 

