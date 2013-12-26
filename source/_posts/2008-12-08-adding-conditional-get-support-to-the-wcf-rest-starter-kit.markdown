---
layout: post
title: "Adding Conditional Get Support to the WCF REST Starter kit"
date: 2008-12-08
comments: true
categories: REST
---

[Some weeks
ago](http://weblogs.asp.net/cibrax/archive/2008/10/06/importance-of-conditional-gets-in-rest.aspx),
I discussed how important "Conditional Get" can be for some scenarios,
specially when we want to make a better use of the network traffic.

The WCF REST Starter kit introduced a new extension "WebCache",
implemented as an operation behavior, to automatically add caching
support to any "Get" operation in a service contract. Using this new
feature is a simple as annotating an existing service operation with a
"WebCache" attribute, as it is shown below:

[WebCache(CacheProfileName = "CacheFor1Min")]

[WebGet(UriTemplate = "customer/{id}")]

[OperationContract]

public Customer GetCustomer(string id)

The underline implementation of this new behavior relies on the existing
ASP.NET web cache. For more details about this new feature, Jesus
Rodriguez has written a very [good summary
here](http://weblogs.asp.net/gsusx/). I also mentioned how to enable Sql
dependencies with this feature [in this
post.](http://weblogs.asp.net/cibrax/archive/2008/11/03/using-sqlcache-dependencies-with-the-new-wcf-webcache-attribute-rest-starter-kit.aspx)

Unfortunately, this implementation does not support Conditional Get out
of the box, which means that the client always receive a complete
response, no matter if the response was served directly from the cache
or not. For instance, if one operation returns a feed with one hundred
items, and the client application does not have a way to specify when
was the last time it got access to that feed, it will have to download
the complete feed all the times.

Although the ASP.NET does provide support for conditional gets through
the public methods "SetLastModified" (for date times) and "SetETag" (for
entity representations), the web cache behavior does not make use of
them.

If we just modified the Web cache behavior to include the following line
at the end, the additional LastModified header will be included in the
response

if (!string.IsNullOrEmpty(cacheProfile.VaryByCustom))

{

   
HttpContext.Current.Response.Cache.SetVaryByCustom(cacheProfile.VaryByCustom);

}

 

**HttpContext.Current.Response.Cache.SetLastModified(DateTime.Now);
//Line Added for Conditional Get support**

It is a simple as that. After enabling some trace to the service, we
will see

Server ASP.NET Development Server/9.0.0.0\
Date Mon, 08 Dec 2008 15:41:54 GMT\
X-AspNet-Version 2.0.50727\
Cache-Control public, max-age=44\
Expires Mon, 08 Dec 2008 15:42:42 GMT\
**Last-Modified Mon, 08 Dec 2008 15:41:43 GMT\
**Vary \*\
Content-Type application/xml; charset=utf-8\
Content-Length 195\
Connection Close

However, setting DateTime.Now for every operation is not efficient for
every scenario because it will be tied to the item lifetime in the
cache. For instance, if our service operation is returning a feed, we
might want to use the feed's last update time.

It will be much better if have a configurable way to set up this date
for every entry in the cache. For that reason, I came up with a solution
based on a simple strategy pattern through a pluggeable interface
"ILastModified",

public interface ILastModified

{

    DateTime? GetLastModifiedForOutput(object[] outputs, object
returnValue);

}

The method "GetLastModifiedForOutput" basically determines the
"LastModified" date for the given outputs parameters and return value of
the service operation. Given this simple interface, we could have
default implementations for feeds or the current date time.

public class SyndicationLastModified : ILastModified

{

    public DateTime? GetLastModifiedForOutput(object[] outputs, object
returnValue)

    {

        if(returnValue is SyndicationFeedFormatter)

        {

            var formatter = (SyndicationFeedFormatter)returnValue;

            return formatter.Feed.LastUpdatedTime.UtcDateTime;

        }

 

        return null;

 

    }

}

 

public class CurrentDateLastModified : ILastModified

{

    public DateTime? GetLastModifiedForOutput(object[] outputs, object
returnValue)

    {

        return DateTime.Now;

    }

}

And finally configure these classes as part of the WebCache attribute.

[WebCache(CacheProfileName = "CacheFor1Min",
**LastModifiedType=typeof(CurrentDateLastModified)**)]

[WebGet(UriTemplate = "customer/{id}")]

[OperationContract]

public Customer GetCustomer(string id)

{

    if (id == "foo")

        return new Customer { Id = id, FullName = "John Foo" };

    else if (id == "bar")

        return new Customer { Id = id, FullName = "John Bar" };

 

    return new Customer { Id = id, FullName = "Unknow" };

}

If no LastModifiedType is specified as argument for the "WebCache"
attribute, the default behavior is used,  resulting in a complete
response message served to the client all the times. Otherwise, the
server will evaluate the "If-Modified-Since" header first to know if the
valid response for that request should be "serving all the content" vs
returning a "Http 304 - Not Modified" status code.

[The sample code is available
here](/images/legacy/WebCache.zip)

