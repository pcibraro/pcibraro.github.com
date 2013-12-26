---
layout: post
title: "Binding form data in ASP.NET Web API"
date: 2012-08-10
comments: true
categories: .NET
---

One scenario that is very common in ASP.NET MVC is to bind form data
(data posted with the media type application/x-www-form-urlencoded)  to
individual parameters or a form collection in a controller action.
However, that scenario does not work quite the same in ASP.NET Web API
as the body content is treated a forward-only stream that can only be
read once. 

If you define an action with multiple simple type arguments such as the
one shown bellow, the model binding runtime will try to map those by
default from the request URI (query string or route data). 

```csharp
public void Post(int id, string value)
{
}
```

That data never comes from the body content unless you decorate the
argument with the [FromBody] attribute. However, only one argument in
the action can be decorated with this attribute, and you get an
exception when you try to decorate more than one. Nevertheless, the
desired effect is far from what you would expect.

If you define an action as follow,

```csharp
public void Post([FromBody]int id)
{
}
```

And you POST a form with the following data,

<http://localhost:64561/api/values>

METHOD: POST

CONTENT-TYPE: application/x-www-form-urlencoded

BODY: id=4

Nothing will be mapped. It does not matter you defined a single
parameter with the same as the one posted in the body.

If you really want to have a single type argument mapped to the body
content, you have to POST a message with the following content

<http://localhost:64561/api/values>

METHOD: POST

CONTENT-TYPE: application/x-www-form-urlencoded

BODY: =4

As you can see, no key defined so the whole is mapped to our argument in
the action. That’s the default behavior for single types. If you want to
change that behavior, the framework is completely extensible and you can
replace the whole model binding infrastructure included out of the box
for one that you find more useful as it is explained
[here](http://blogs.msdn.com/b/jmstall/archive/2012/04/18/mvc-style-parameter-binding-for-webapi.aspx)
by Mike Stalls (which shows how you can replicate the same model used by
ASP.NET MVC).

For complex types, the framework includes two MediaTypeFormatter
implementations. The first one is FormUrlEncodedMediaTypeFormatter,
which knows how to map every key/value in the form data to a
FormDataCollection or a JTokenType (a dynamic type for expressing JSON
available as part of the Json.NET library). These two are useful when
you want to get access to all the values in form in a generic manner.
You define for example a controller that receives a FormDataCollection
instance like this,

```csharp
public void Post(FormDataCollection form)
{
}
```

The second one is JQueryMvcFormUrlEncodedFormatter, which derives from
the first one and knows how to map the form data to a concrete type
representing a model. For example, a model defined as follow will match
a body content with the keys “Id” and “Value”

```csharp
public class MyModel
{
   public int Id { get; set; }
   public string Value { get; set; }
}
```

Therefore, your controller action can either receive a generic form
collection (or a JTokenType) or a complex model type as any of these two
will handle the deserialization of the body content type into the
expected model.

