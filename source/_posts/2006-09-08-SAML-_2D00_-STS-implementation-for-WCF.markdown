---
layout: post
title: "SAML - STS implementation for WCF"
date: 2006-09-08
comments: true
categories: WCF
---

I finally decided to publish a STS implementation for WCF.  (It is based
on one of my previous posts, ["Implementing a Secure token service with
WCF"](/cibrax/archive/2006/03/14/440222.aspx))

I received a lot of mails from people asking me for a official version
of that code, so here it
[is](/images/legacy/STSExample.zip). I implemented this
STS using latest WCF release (September CTP).

All the classes previously provided by WCF to parse the RST and RSTR
messages have gone (They are now private). I am not sure about the
reason to take that decision, but it certainly complicates everything.

Therefore, a person developing a STS (me) has only two options:

1.  Create different classes to parse those messages. In other words,
    duplicate all the code previously provided by WCF
2.  Copy that code from the [Martin Gudgin's
    implementation](http://pluralsight.com/blogs/mgudgin/archive/2006/06/19/28503.aspx)
    (Same as 1, but it makes your life easier).  

The code also includes the following features:

-   A custom Security Token Manager to reuse the same SAML token
    instance in different WCF channels. You can find more information
    about this solution in this
    [post](/cibrax/archive/2006/03/27/441227.aspx). This token manager
    is quite simple, and it only shows how this task can be performed. 
-   Two configuration files showing different binding settings between
    the client, the service and the STS. The first one uses the standar
    wsFederationHttpBinding, and the last one uses customBindings.

Download the code from this
[location](/images/legacy/STSExample.zip)

Update, feedback received from the WCF team regarding the RST/RSTR
classes:

*"We made those classes internal during our RC1 milestone because this
will enable us to align the product better for future version of the
WS-Trust protocol without exposing user to the necessary changes. When
doing this we have actually changed our samples that show our Federation
feature so that those samples now contain source code of RST/RSTR
library that you can use to parse RST and RSTR messages in your
application. For example the FederationHttp binding sample in the
Windows SDK in WCF samples collection contains source code for parsing
RST/RSTR messages. It can be found in
TechnologySamples\\Basic\\Binding\\WS\\FederationHttp directory. I hope
this answers your concern. Thanks, --Jan" *\
 

 

