---
layout: post
title: "Custom domains for Azure Hosted Services"
date: 2011-10-05
comments: true
categories: .NET
---

A hosted service in azure is typically assigned with two public
addresses, one for the production environment with a DNS name ending in
cloudapp.net such as [your name].cloudapp.net and an auto generated DNS
name for the staging environment such as
4969aae4e18f4699aa88223e1e73ba8e.cloudapp.net. There are multiple
reasons you might want to map your custom domain name to these public
names in Azure, but these are the most common ones I can imagine,

-   You don’t want to tie your websites to a single cloud hosting vendor
    such as Microsoft
-   You host multiple websites in a single hosted service in azure, so a
    single address such as xxx.cloudapp.net  does not make sense
    anymore. In that scenario, you need to use different port numbers
    for the websites or use host headers, which is the typical solution.
    For example, you might want to map [your name].com and [your other
    name].com to two different websites in the same hosted service
    (xxx.cloudapp.net).
-   You want to map a single domain to multiple hosted services in azure
    in different regions, and use geographic DNS routing to balance
    traffic. For example, you might have xxx-us.cloudapp.net for a
    hosted service in the US and xxx-eu.cloudapp.net for another hosted
    service in Europe but a single domain [your name].com that
    automatically routes the traffic according to the source of the http
    requests.

In the two first cases, the use of CName records in a domain that you
own will work for redirecting all the traffic from there to the hosted
service in Azure. The last one might require an additional DNS service
like the Traffic Manager in Azure or Route53 in Amazon. 

It’s common nowadays for most websites to host two different versions of
the same portal for mobile devices and regular browsers, which basically
fits in the scenario I described as number \#2. We can not use the
default addressing schema in Azure anymore because the IIS instance
running in hosted service wouldn’t know how to route the traffic to the
right portal. However, this can be resolved with a combination of CName
records at DNS level and host headers at IIS configuration level.

Configuring different host headers for the websites in IIS is easy to
achieve with the use of multiples bindings and endpoints at
configuration level.

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: white; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px"}
<Sites>
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px"}
  <Site name="Mobile" physicalDirectory="..\xxx">
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: white; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px"}
    <Bindings>
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px"}
      <Binding name="HttpIn" endpointName="HttpIn" hostHeader="m.xxx.com" />
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: white; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px"}
      <Binding name="HttpInStaging" endpointName="HttpIn" hostHeader="stagingm.xxx.com" />          
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px"}
    </Bindings>
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: white; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px"}
  </Site>
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px"}
  <Site name="Portal" physicalDirectory="..\xxx">
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: white; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px"}
    <Bindings>
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px"}
      <Binding name="HttpIn" endpointName="HttpIn" hostHeader="www.xxx.com" />      
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: white; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px"}
      <Binding name="HttpInStaging" endpointName="HttpIn" hostHeader="staging.xxx.com" />
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px"}
    </Bindings>
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: white; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px"}
  </Site>
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px"}
</Sites>
~~~~

The snippet above illustrates how the Service Definition should look for
supporting two different web sites for the mobile site and regular web
site. An important thing to notice in the configuration is that I also
included the host headers for staging, so you can test staging as well
before making the switch to production. This means will require a
different set of CName records for staging and production,

-   The “www” and “m” records will point to the production website in
    azure, xxx.cloudapp.net
-   The “staging” and “stagingm” records will point to the auto
    generated address for staging in azure

A CName record for the auto generated address in staging is not a good
thing as it gets generated every time you create a new staging
deployment. However, there is no other good way to fix in the meantime
unless you actually do not include this definition for staging and you
test everything local by pointing your windows host file to the IP
address in staging (For instance, [www.xxx.com](http://www.xxx.com)
[Staging IP Address] and m.xxx.com [Staging IP Address]).

The service definition can not be changed or updated once the deployment
has been completed so make sure to get it right in the first place.

