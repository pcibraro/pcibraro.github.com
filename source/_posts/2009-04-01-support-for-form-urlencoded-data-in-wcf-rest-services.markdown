---
layout: post
title: "Support for Form-UrlEncoded data in WCF Rest services"
date: 2009-04-01
comments: true
categories: REST
---

Some people have been asking around in the MSDN forums or
[stackoverflow](http://stackoverflow.com/questions/604463/best-way-to-support-application-x-www-form-urlencoded-post-data-with-wcf)
about a possible way to pass form-urlencoded data to an existing WCF
REST service. Although this is not a core feature in WCF from my point
view, there is still some support for addressing this scenario in the
REST Starter kit.

This feature takes the form of a new message formatter,
FormsPostDispatchMessageFormatter, which is automatically injected to
any existing service hosted within the new WebServiceHost2 host (shipped
as part of the kit)

This formatter basically intercepts all messages, and handle those that
contains “application/x-www-form-urlencoded” as content type in the
request header. The rest of the messages are handled by the existing
formatters as usual.

The service operation must expose a NameValueCollection argument in the
method signature in order to receive the data. This collection will
contain the key/value pairs that were parsed from request body.

For example,

[WebInvoke(UriTemplate = "SetData")]

[OperationContract]

void SetData(NameValueCollection formsData)

{

    this.value = Int32.Parse(formsData["counter"]);

}

“Counter”  in this case represents a field in the form that was posted
to the service.

\<

FORM action="service.svc/SetData" method="post"\>

Input Counter value and hit enter:

\<INPUT type="text" name="Counter"\>

\</FORM\>

