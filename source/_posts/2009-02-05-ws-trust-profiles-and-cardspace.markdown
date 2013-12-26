---
layout: post
title: "WS-TRUST profiles and Cardspace"
date: 2009-02-05
comments: true
categories: Cardspace
---

Geneva framework supports today the two WS-Trust profiles, Active and
Passive.

The active profile deals specially with applications that are able to
make soap request to any WS-Trust endpoint. On other hand, the passive
profile is for clients that are unable to emit proper SOAP (a web
browser for instance) and therefore receive the name of "passive
requestors". This last one involves browser-based communication with
several http redirects between the different parties (client, STS and
relying party).

Cardspace embedded in a web browser page however is not a Passive
client. Once the user decides to be authenticated in a website with an
information card, the Cardspace identity selector will negotiate and get
the issue token from the identity provider using the active profile.
Finally, the identity provider will pass the token to the browser using
some Inter-Process communication, and the browser can later submit the
token to the server using an standard http mechanism like a web post.

As you can see, Carspace in a browser is actually an hybrid between
Active and Passive.
[Vittorio](http://blogs.msdn.com/vbertocci/archive/2008/06/05/active-passive-and-passive-aggressive.aspx)
has also discussed this scenario in the past, he called it
"Passive-Aggressive".

 

