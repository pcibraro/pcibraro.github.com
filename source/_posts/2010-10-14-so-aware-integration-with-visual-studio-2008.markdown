---
layout: post
title: "SO-Aware integration with Visual Studio 2008"
date: 2010-10-14
comments: true
categories: .NET
---

I am happy to announce today the support in Visual Studio 2008 for
adding new service references from the SO-Aware service repository. We
have created a simple plugin that you can register in Visual Studio to
support this new functionality.

The functionality provided by this plugin is equivalent to what you
already have for Visual Studio 2010. A new option is added to the
project context menu “Add Service Reference From SO-Aware”, which is
almost similar to the classic “Add Service Reference” option with the
difference that you can look for existing services in the repository and
the generated proxy derives from ConfigurableClientBase\<T\> rather than
ClientBase\<T\>

![AddServiceReference](http://weblogs.asp.net/blogs/cibrax/AddServiceReference_0B890770.png "AddServiceReference")

Once you clicked in that option, a new screen will appear to select one
of the existing services in the repository.

![Repository](http://weblogs.asp.net/blogs/cibrax/Repository_0F46AC40.png "Repository")

The generated proxy will have methods to consume the service operation
as any regular service proxy, but also two methods that are specific to
WCF for optionally configuring a client behavior (SetClientBehavior) or
a binding (SetDefaultBinding) from the repository. These are optional as
otherwise, the behaviors and bindings configured for the service in the
repository will be used.

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: white; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
CustomersClient client = new CustomersClient();
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
client.SetClientBehavior("BehaviorName");
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: white; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
client.SetDefaultBinding("BindingName");
~~~~

Those methods receive an string with the name of the behavior or binding
you want to resolve from the repository, so there is no need to have the
“serviceModel” configuration section anymore :).

This new plugin is available as part of the SO-Aware SDK that you can
get from
[here](http://www.tellagostudios.com/resources/so-aware-sdk-samples-and-utilities).

