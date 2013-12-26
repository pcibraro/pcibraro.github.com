---
layout: post
title: "SWUtil - A new tool for generating service proxies from the SO-Aware repository."
date: 2010-09-15
comments: true
categories: .NET
---

As we announced last week, we are shipping a new Visual Studio plugin
for generating service proxies as part of the SO-Aware SDK. The
functionality is equivalent to what you find today in the “Add Service
Reference” command, but the results are much better as you get a proxy
that does not require any WCF configuration, and also knows how to
resolve bindings and behaviors from the repository.

However, that plugin is only available for Visual Studio 2010, meaning
that you need an alternative solution for generating the same equivalent
proxy if you are using older versions of Visual Studio or you are not
even using this development environment. Here is where “swutil.exe”
comes to fill that gap.

This tool is equivalent to “svcutil.exe”, the one that comes with the
.NET framework for generating WCF service proxies, but the result is a
much more intelligent proxy that does not require any previous knowledge
of WCF configuration.

The following arguments are supported by this new tool,

swutil.exe -help

Parameters:

-help           Prints the help screen. \
-uri             Service Repository Uri \
-version       Service Version Name \
-category    Configuration Category \
-out            Output file \
-language    Language: cs or vb \
-serializer    xml or datacontract

The generated proxy derives from a specific SO-Aware base class
“ConfigurableClientBase\<T\>”, and not the traditional one
“ClientBase\<T\>” that you find in the WCF.

This specific base class gives you access to some methods like
“SetClientBehavior” or “SetDefaultBinding” that become handy for
automatically resolving bindings and behavior configuration from the
repository.

Those methods receive an string with the “binding” or “behavior” name,
and automatically inject the equivalent WCF object into the WCF channel
after having resolved that name in the repository.

The following lines illustrate how you would use this new proxy to
consume a service.

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: white; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
EchoClaimsClient proxy = new EchoClaimsClient(ServiceUri, "EchoClaims(1.0)", null, "CustomBinding_IEchoClaims");
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
proxy.SetClientBehavior("echoClaimsEndpointBehavior");
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: white; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
proxy.SetDefaultBinding("echoClaims_MutualCertificate");
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
 
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: white; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
var response = proxy.Echo();
~~~~

That’s all, no configuration is required on the client side for
consuming the service. All the magic happens in that proxy class :)

