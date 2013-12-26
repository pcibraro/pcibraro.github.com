---
layout: post
title: "A Bad Idea, EF Entities over WCF"
date: 2009-06-25
comments: true
categories: .NET
---

Do you want to see something very ugly ?. Try to send a complete EF
entity directly as a data contract over the wire with WCF,

\<Order
xmlns:i="[http://www.w3.org/2001/XMLSchema-instance](http://www.w3.org/2001/XMLSchema-instance)"
z:Id="i1"
xmlns:z="[http://schemas.microsoft.com/2003/10/Serialization/](http://schemas.microsoft.com/2003/10/Serialization/)"
xmlns="[http://schemas.datacontract.org/2004/07/ClassLibrary2](http://schemas.datacontract.org/2004/07/ClassLibrary2)"\>
\
  \<EntityKey
xmlns:d2p1="[http://schemas.datacontract.org/2004/07/System.Data](http://schemas.datacontract.org/2004/07/System.Data)"
i:nil="true"
xmlns="[http://schemas.datacontract.org/2004/07/System.Data.Objects.DataClasses](http://schemas.datacontract.org/2004/07/System.Data.Objects.DataClasses)"
/\> \
  \<Description\>My new order\</Description\> \
  \<OrderId\>1\</OrderId\> \
  \<OrderItem\> \
    \<OrderItem z:Id="i2"\> \
      \<EntityKey
xmlns:d4p1="[http://schemas.datacontract.org/2004/07/System.Data](http://schemas.datacontract.org/2004/07/System.Data)"
i:nil="true"
xmlns="[http://schemas.datacontract.org/2004/07/System.Data.Objects.DataClasses](http://schemas.datacontract.org/2004/07/System.Data.Objects.DataClasses)"
/\> \
      \<Currency\>USD\</Currency\> \
      \<Description\>item 1\</Description\> \
      \<ItemId\>1\</ItemId\> \
      \<Order z:Ref="i1" /\> \
      \<OrderReference
xmlns:d4p1="[http://schemas.datacontract.org/2004/07/System.Data.Objects.DataClasses](http://schemas.datacontract.org/2004/07/System.Data.Objects.DataClasses)"\>
\
        \<d4p1:EntityKey
xmlns:d5p1="[http://schemas.datacontract.org/2004/07/System.Data](http://schemas.datacontract.org/2004/07/System.Data)"
i:nil="true" /\> \
      \</OrderReference\> \
      \<UnitPrice\>10\</UnitPrice\> \
    \</OrderItem\> \
    \<OrderItem z:Id="i3"\> \
      \<EntityKey
xmlns:d4p1="[http://schemas.datacontract.org/2004/07/System.Data](http://schemas.datacontract.org/2004/07/System.Data)"
i:nil="true"
xmlns="[http://schemas.datacontract.org/2004/07/System.Data.Objects.DataClasses](http://schemas.datacontract.org/2004/07/System.Data.Objects.DataClasses)"
/\> \
      \<Currency\>USD\</Currency\> \
      \<Description\>item 2\</Description\> \
      \<ItemId\>2\</ItemId\> \
      \<Order z:Ref="i1" /\> \
      \<OrderReference
xmlns:d4p1="[http://schemas.datacontract.org/2004/07/System.Data.Objects.DataClasses](http://schemas.datacontract.org/2004/07/System.Data.Objects.DataClasses)"\>
\
        \<d4p1:EntityKey
xmlns:d5p1="[http://schemas.datacontract.org/2004/07/System.Data](http://schemas.datacontract.org/2004/07/System.Data)"
i:nil="true" /\> \
      \</OrderReference\> \
      \<UnitPrice\>145\</UnitPrice\> \
    \</OrderItem\> \
  \</OrderItem\> \
\</Order\>

That is the result of serializing a complete object graph. I do not know
what the folks in Microsoft were thinking when they decided to enable a
feature like this. They made a good work teaching us about how evil
Datasets were for interoperability with other platforms, and now they
came up with a solution like this, no way.

I always like the idea of making boundaries explicit, that is the
approach the WCF team took with the first version of the framework
(Every data contract should be decorated with DataContract and
DataMember attributes).  If you break that rule, you will end up with
things like this.

I know this a solution for people that want to develop things quickly
with no need to create DTOs, and do all the mapping stuff, but there are
good tools and frameworks like
[AutoMapper](http://www.codeplex.com/AutoMapper) to help with that. Or
you can use ADO.NET Data Services today, which is also prepared to
handle this kind of scenarios, all the transformations between layers
are made by the framework itself, and still you get a very nice RESTful
api. We still need some more features in ADO.NET DS to have a more
complete framework, there is not good support today for injecting
business logic, [it can be
done](http://weblogs.asp.net/cibrax/archive/2009/06/08/injecting-custom-logic-in-ado-net-data-services.aspx)
but it is complicated, it looks like the [next
version](http://johnpapa.net/silverlight/ado-net-data-services-v1-5-is-on-its-way/)
will address that aspect better.

