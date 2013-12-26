---
layout: post
title: "Building Hypermedia Web APIs"
date: 2012-03-06
comments: true
categories: .NET
---

Hypermedia is one of those concepts really hard to grasp when building
Http aware APIs (or Web API’s). As human beings, we are constantly
dealing with hypermedia in the existing web by following links or
posting data from some forms that take us to a next level.

We typically remember the entry point or URL for the retrieving the home
page of web site, and we can move from there on to different sections
using hypermedia artifacts. Those URL usually tend to be nice and easy
to remember, but they don’t have to be as it is not something http by
itself mandates. We could rely on a search engine like Google or Bing to
find those URLs for us. You can even use some cryptographic URLs for the
web pages in your website, and still provide a nice experience for the
user by proving the right links to browse the content.

When you build Http API’s for being consumed for other systems, the
history is not any different. There is no any reason for client
applications to remember all the possible resources and their location
(URLs) if the server can provide those. Hardcoding URL’s on the client
side is a bad thing for two main reasons,

1.  The server can change the location for an specific resource, so all
    the clients having a hardcoded URL for that resource will break.
2.  It pushes some knowledge about the application state workflow to the
    client side. What happens if an action in a resource is only
    available for a given state, shall we put that logic in any possible
    API consumer ?. Definitely not, the server should always mandate
    what can be done or not with a resource. For example, if the state
    of a purchase order is canceled, the client application application
    shouldn’t be allowed to submit that PO. If we have an user in front
    of an UI using that API, he shouldn’t see the submit button enabled
    (That logic for enabling or disabling the button could be driven by
    the server using links).

This is one of the gray areas that typically differentiates a regular
Web API’s from a RESTful API, but there are some other constraints that
also applies, so this discussing about RESTful services probably don’t
make sense in most cases. What matters in the end is that the API uses
HTTP correctly as application protocol and leverage hypermedia when it
is possible. By enabling Hypermedia, you can create self-discoverable
APIs, which are not an excuse for not providing documentation as I
usually hear, but are more flexible in terms of updatability.

Many of the media types we use nowadays for building Web API’s such as
JSON or XML don’t have a built-in concept for representing hypermedia
links as HTML does with links and forms. You can leverage those media
types by defining a way to express hypermedia, but again, it requires
client to understand how the hypermedia semantics are defined on top of
those. Other media types like XHtml or ATOM already support some of the
hypermedia concepts like links or forms.

By moving forward with this idea of Hypermedia API on top of JSON or
XML, Mike Kelly wrote a draft for an specific media type called
[“HAL”](http://stateless.co/hal_specification.html).   HAL extends JSON
and XML with all the semantics required to express resources and their
links. This fragment from the spec draft illustrates how an order can be
modeled using the xml variant of HAL,

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
<resource href="/orders">
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
  <link rel="next" href="/orders?page=2" />
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
  <link rel="search" href="/orders?id={order_id}" />
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
  <resource rel="order" href="/orders/123">
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
    <link rel="customer" href="/customer/bob" title="Bob Jones &lt;bob@jones.com>" />
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
    <resource rel="basket" href="/orders/123/basket">
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
      <item>
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
        <sku>ABC123</sku>
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
        <quantity>2</sku>
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
        <price>9.50</price>
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
      </item>
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
      <item>
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
        <sku>GFZ111</sku>
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
        <quantity>1</quantity>
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
        <price>11.00</price>
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
      </item>
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
    </resource>
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
    <total>30.00</total>
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
    <currency>USD</currency>
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
    <status>shipped</status>
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
    <placed>2011-01-16</placed>
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
  </resource>
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
  <resource rel="order" href="/orders/124">
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
    <link rel="customer" href="/customer/jen" title="Jen Harris &lt;jen@internet.com>" />    
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
    <resource rel="basket" href="/orders/124/basket">
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
      <item>
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
        <sku>KLM222</sku>
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
        <quantity>1</sku>
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
        <price>9.00</price>
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
      </item>
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
      <item>
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
        <sku>HHI50</sku>
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
        <quantity>1</quantity>
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
        <price>11.00</price>
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
      </item>
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
    </resource>
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
    <total>20.00</total>
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
    <currency>USD</currency<status>processing</status>
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
    <placed>2011-01-16</placed>
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
  </resource>
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
</resource>
~~~~

You can see the different entities in your system can be modeled and
represented as resources, which are linked, and attributes like “rel” or
“href” are used to express the role of the entity and it’s location.
Steve Michelotti created an specific formatter for HAL in WCF Web API
available [here](https://bitbucket.org/smichelotti/hal-media-type), but
it hasn’t be updated to the latest ASP.NET Web API version yet.

I have been able to see two different kinds of hypermedia APIs over the
years. APIs returning links for representing state transitions or linked
resources that put some out of band assumptions on the consumer for how
the transition should be executed. For example, consider the following
example with the representation of a PO.

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
<purchaseOrder>
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
  <id>90</id>
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
  <sku>AJJ34</sku>  
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
  <link rel="customer" href="/customers/foo"/>
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
  <link rel="approve" href="/po/90/approve"/>
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
  <link rel="cancel" href="/po/90/cancel"/>
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
</purchaseOrder>
~~~~

The representation contains a link for retrieving the representation of
the associated customer (which assumes the client should send an HTTP to
that URL for getting the representation), and two additional links for
approving or canceling the PO (which also assumes the client knows how
to post a message to those URLs). You see, the consumer is still coupled
to the Web API implementation with some out of band details, but still
is self-discoverable and the consumer can determine which actions can be
executed over that resource (See the associated customer details,
approve it or cancel it).

There are also other kind of APIs that use Forms as you would find them
in the HTML or XHTML media types. Good part of these APIs are driven by
forms submissions. Let’s see an example to illustrate the concept.

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
<ul id="purchaseOrder">
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
  <li id="id">90</li>
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
  <li id="sku">AJJ34</li>  
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
</ul>
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
<a href="/customers/foo" rel="customer"/>
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
<a href="/po/90/approve_form" rel="approve"/>
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
<a href="/po/90/cancel_form" rel="cancel"/>
~~~~

The purchase order is returned this time as an XHTML representation and
contains links which basically supports the HTTP GET semantics. An HTTP
GET to the “customer” link would return the associated customer
representation. However, an HTTP GET to the “approve” or “cancel” links
would return an http form this time for approving or canceling the
order.

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
<form action="/po/90/approve" method="POST">
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
  <input type="hidden" id="id">90</input>
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
  <input type="hidden" id="___forgeryToken">XXXXXXXX</input>
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
</form>
~~~~

The consumer now has been decoupled of certain details for approving the
order for example. It only needs to submit this form using an HTTP POST
to the URL specified in the action attribute. The server can also
includes additional information in the form as a forgery token for
example to avoid “Cross-site Request Forgery” (CSRF) attacks.

