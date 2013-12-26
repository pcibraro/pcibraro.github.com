---
layout: post
title: "Describing RESTful applications"
date: 2009-01-08
comments: true
categories: 
---

The other day I came across [this interesting
article](http://www.infoq.com/articles/subbu-allamaraju-rest) written by
[Subbu Allamaraju](http://www.subbu.org/), where he discusses some of
the aspects that drive the design of a pure and self-describing RESTful
interface for a service (Or service contract in other words).

He points out three main pieces that every service contract should have:

​1. An uniform interface. An uniform interface is probably one the
aspects that we best know about REST, or we may heard often when a REST
service is described. This refers to the fact that each service exposes
the same interface, which is a good thing for eliminating ad hoc
messages and focusing primarily on an standard API that defines pieces
of information that can be retrieved and manipulated. Instead of adding
special function calls or interfaces to the architecture, new services
add new pieces of information can be manipulated using standard
requests. For instance, in Http RESTful services, this point involves
using standard http verbs, headers and status codes.

​2. Resource representations based on media types. This is quite
interesting, what he proposes is to use a more specialized
"content-type" for any of the xml schemas that a service may return. He
confirms that generic media types such as "application/xml" are not
enough to distinguish resource representations. We might want to know if
the resource represents an account or a customer without making any
assumptions about the resource's URI. In those cases, content types like
"application/vnd.bank.org.accounts+xml" or
"application/vnd.bank.org.customers+xml" would be more helpful and still
being valid according to the spec  "rfc3023, XML media types".

200 OK \
**Content-Type: application/vnd.bank.org.account+xml**

\<accounts xmlns="urn:org:bank:accounts"\> \
    \<account\> \
        \<id\>AZA12093\</id\>        \
        \<balance currency="USD"\>993.95\</balance\> \
    \</account\> \
\</accounts\>

​3. Contextual links to resources. In addition to the resource
representation, the client should also receive links representing what
it can do next (Taking the form of a workflow). As Subbu concludes, this
aspect decouples the client from the actual URIs of resources, and the
client does not need to know the rest of the URIs until run-time.

200 OK \
Content-Type: application/vnd.bank.org.account+xml;charset=UTF-8

\<accounts xmlns="urn:org:bank:accounts"\> \
    \<account\> \
        \<id\>AZA12093\</id\> \
        **\<link
href="**[**http://bank.org/account/AZA12093**](http://bank.org/account/AZA12093)**"
rel="self"/\> \
        \<link
rel="**[**http://bank.org/rel/transfer**](http://bank.org/rel/transfer)**edit"
\
              type="application/vnd.bank.org.transfer+xml" \
             
href="**[**http://bank.org/transfers"/**](http://bank.org/transfers%22/)**\>
\
        \<link
rel="**[**http://bank.org/rel/customer**](http://bank.org/rel/customer)**"
\
              type="application/vnd.bank.org.customer+xml" \
             
href="**[**http://bank.org/customer/7t676323a"/**](http://bank.org/customer/7t676323a%22/)**\>
\
**        \<balance currency="USD"\>993.95\</balance\> \
    \</account\> \
    .....

As you can see in the example above, the account representation contains
links to itself (Self link) and other resources. The operations that can
be performed over those resources are expressed by the "rel" attribute.

Altough there is not built-in support in WCF to enforce these aspects
automatically, it can be easily done in the service implementation using
specific data contracts and the WebOperationContext. 

Setting up the content type before returning the service response:

public Account GetAccount(string id)

{

  WebOperationContext.Current.OutgoingResponse.ContentType =
"application/vnd.bank.org.account+xml";

  return repository.Accounts.Where(a =\> a.Id == id).FirstOrDefault();

}

A link can be modeled as a regular data contract:

[DataContract(Name="link")]

public class Link

{

   [DataMember(Name="href")]

   public string Href { get; set; }

 

   [DataMember(Name = "rel")]

   public string Rel { get; set; }

 

   [DataMember(Name = "type")]

   public string Type { get; set; }

}

Read the [article](http://www.infoq.com/articles/subbu-allamaraju-rest)
for more information.

