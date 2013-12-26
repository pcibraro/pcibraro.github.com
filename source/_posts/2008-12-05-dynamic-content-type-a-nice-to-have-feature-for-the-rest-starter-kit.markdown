---
layout: post
title: "Dynamic Content Type - A nice to have feature for the WCF REST Starter Kit"
date: 2008-12-05
comments: true
categories: REST
---

One of the missing features in the WCF web model is the ability to have
a single operation definition and switch the content type for the
response according to some runtime setting (Which could be the Accept or
Content-Type request header for instance).

Today, I came across this post "[WCF REST Services: Setting the response
format based on request's expected
type](http://damianblog.com/2008/10/31/wcf-rest-dynamic-response/)" that
shows exactly that. The solution is very clean and based on some
extensions to the existing WebHttpBehavior and WebHttpHost classes.

Kyle Beyer took a similar approach to switch the context type of the
request messages as well. His implementation was layered on top of
existing classes in the REST Starter kit, and it can find
[here](http://daptivate.com/archive/2008/11/16/wcf-and-rest-an-approach-to-using-the-content-type-and-accept-http-headers-for-object-serialization.aspx).

Both implementations inspect the "Content-Type" header of the request
messages to determine the WCF formatter to use. What it would very nice
to have in the Starter kit is the possibility to switch the content type
dynamically not only by the "Content-Type" header, but also by some
other settings like UriTemplates or QueryString arguments.

It wouldn't be nice to have an attribute with the following syntax:

[WebHelp(Comment = "Returns the items in the collection in the format
specified by the Accept header, along with URI links to each item.")]

[WebGet(UriTemplate = CollectionServiceBase\<TItem\>.ItemsUriTemplate)]

[OperationContract]

[DynamicContentType(Direction=DynamicDirection.Both,
Resolution=DynamicResolution.QueryString,
QueryStringArgument="content")]

ItemInfoList\<TItem\> GetItems();

Where Direction and Resolution could support some of these values,

public enum Direction

{

    RequestOnly,

    ResponseOnly,

    Both

}

 

public enum Resolution

{

    ContentTypeHeader,

    UriTemplate,

    QueryString

}

 

With that attribute in place, it will not be necessary anymore to
redefine the same operation many times just to change the content type.
For instance, the following service contract can be simplified with the
use of this new attribute,

[OperationContract]

ItemInfoList\<TItem\> GetItemsInXml();

 

[WebGet(UriTemplate = CollectionServiceBase\<TItem\>.JsonItemsTemplate,
ResponseFormat = WebMessageFormat.Json)]

[OperationContract]

ItemInfoList\<TItem\> GetItemsInJson();

I will see if I have some time to implement a prototype using that
syntax for the DynamicContentType attribute.

In the meantime, I think it would be great to have support out of the
box for a feature like this in the WCF REST Starter kit, what's your
opinion ?

