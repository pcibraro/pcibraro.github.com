---
layout: post
title: "Windows Live Contacts API"
date: 2008-04-09
comments: true
categories: Web-Services
---

With the boom in social networking, many web sites have started offering
new tools for building or expanding your initial network of contacts.

For instance, sites like Facebook or Linkedin provide a tool to get your
Windows Live contacts and use them to more easily find or invite those
people into your social networks.

Although the idea is very good, the current implementation contains some
serious security issues from my point of view.  The user has to enter in
some way his Windows LiveID credentials into those sites (using a custom
http form), so they can log into Windows Live and get the user's
contacts.

There is not any particular difference between this approach and a
phising web site created by a malicious user only with the intention of
getting your personal information.

Not all people (including me) trust these sites enough to provide them
with valuable Windows LiveID credentials. These sites are very
well-known, but we are not certainly sure which will be the final user
of our credentials. (or even, if they are keeping them somewhere).

This security issue could be basically solved if Windows Live provided
two fundamental things:

1.  Http REST Services to get the user contacts or other personal
information.

​2. An authentication mechanism for those services based on security
tokens. Something similar to what [OAuth](http://oauth.net/) provides,
so the user will never have to enter his credentials in a site different
from Microsoft again. Another advantage of this protocol is that the
user will finally decide whether he authorize third party sites to get
his personal information or not. If you are curious about how OAuth
works behind scene,  an excellent "Begginer's guide" is available
[here](http://www.hueniverse.com/hueniverse/2007/10/beginners-gui-1.html).

Fortunately, the Windows Live team has made an excellent progress in
these two aspects. One one hand, they have developed an "[Windows Live
ID Delegated Authentication
SDK](http://msdn2.microsoft.com/en-us/library/cc287637.aspx)" to
integrate Application providers through a protocol pretty similar to
OAuth (I haven't had enough time yet to take a more detailed look at
this SDK)

From the MSDN site,

"*Delegated Authentication is based on a block of information, called a
consent token, that is provided to your Web site by the Windows Live ID
service for a given resource provider (such as contacts and photos). To
obtain a consent token for use at a particular resource provider, you
must first request it from the user by means of the Windows Live ID
consent service. Your application must then manage the authentication
data that is returned. For detailed information about how to request and
manage consent, see the*[*Windows Live Delegated Authentication
SDK*](http://msdn.microsoft.com/en-us/library/cc287637.aspx)."

On other hand, as part of "[Windows Live User Data
APIs](http://msdn2.microsoft.com/en-us/library/cc305075.aspx)", they
have started providing REST services to allow Windows Live users to
safely and securely share their information stored in Windows Live
services. One of this services is what they have called "[Windows Live
Contacts API](http://msdn2.microsoft.com/en-us/library/bb463989.aspx)",
a REST service that enables developers to programmatically submit
queries to, and retrieve results from, the Windows Live Contacts Address
Book database service.

Hopefully, it will be a matter of time until [Facebook or Linkedin start
integrating](http://dev.live.com/blogs/devlive/archive/2008/03/25/237.aspx)
services like these into their sites for the benefit of all :).

