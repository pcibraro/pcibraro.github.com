---
layout: post
title: "Validating your models in ASP.NET Web API"
date: 2012-02-23
comments: true
categories: .NET
---

One of the nice things about having a single extensibility model between
ASP.NET MVC and Web API is that you can get many of the great MVC
features for free. Model binding and validation is one of them.

A simple action in Web API controller is typically associated to a Http
verb and can optionally receive or return a model (or a message if you
want to have better control over the http messaging details). For
example, the following implementation illustrates a simple case for
creating a contact when an Http POST is received,

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
public class ContactController : ApiController
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
{
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
    IContactRepository repository;
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
 
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
    public ContactController(IContactRepository repository)
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
    {
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
        this.repository = repository;
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
    }
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
 
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
    public HttpResponseMessage Post(Contact contact)
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
    {
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
        int id = this.repository.Create(contact);
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
 
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
        var response = new HttpResponseMessage(HttpStatusCode.Created);
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
        response.Headers.Location = new Uri("/api/Contact/" + id);
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
 
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
        return response;
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
    }
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
}
~~~~

All the details for deserializing the request message into a Contact
model is automatically handled by the Web API infrastructure. It
actually uses the model binding feature found in MVC but it also adds
content negotiation on top of it.

Content negotiation in Web API is implemented through formatters that
know how to serialize/deserialize an specific payload associated to a
content type (json or html for example) into a model.

As you probably know, the model binding infrastructure in MVC also
allows validations on model classes decorated with data annotations.
That means you can decorate your models with different validation
attributes as it shown bellow,

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
public class Contact
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
{
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
    [Required]
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
    public string FullName { get; set; }
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
 
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
    [Email]
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
    public string Email { get; set; }
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
}
~~~~

The model binding infrastructure will validate the model but Web API
will not do anything with it by default. If you want to reject a message
based on the result of the validations, you need to implement a custom
filter in the current bits.

That can be done by extending an ActionFilterAttribute and overriding
the OnActionExecuting method.

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
public class ValidationActionFilter : ActionFilterAttribute
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
{
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
    public override void OnActionExecuting(System.Web.Http.Controllers.HttpActionContext actionContext)
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
    {
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
        if (!actionContext.ModelState.IsValid)
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
        {
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
            var errors = actionContext.ModelState
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
                .Where(e => e.Value.Errors.Count > 0)
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
                .Select(e => new Error
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
                {
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
                    Name = e.Key,
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
                    Message = e.Value.Errors.First().ErrorMessage
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
                }).ToArray();
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
 
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
            actionContext.Response = new HttpResponseMessage<Error[]>(errors, HttpStatusCode.BadRequest);
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
        }
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
    }
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
}
~~~~

As you can see, the implementation is very simple. It only checks
whether the model state is valid and return a list of validation errors.
The execution pipeline will be automatically interrupted and response
will be sent back to the client with the list of errors. I am also using
a model for representing an error so Web API can send the expected wire
format to the client application (If the client is expecting json, it
will receive a list of errors formatted as json for example).

This validation attribute can be injecting into the execution pipeline
by either decorating the api controller with it or by adding it to the
global filters collection.

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
public static void RegisterGlobalFilters(GlobalFilterCollection filters)
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
{
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
    filters.Add(new HandleErrorAttribute());
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
    filters.Add(new ValidationActionFilter());
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
}
~~~~

Now, let’s see how this works in action with Fiddler. If we send a json
message with an invalid email,

****

**POST http://localhost:16913/api/Contact HTTP/1.1**

**Content-Type: application/json**

**Host: localhost:16913**

**Content-Length: 30**

**{"fullname":"re","email":"re"}**

We will receive a response also formatted as json,

****

**HTTP/1.1 400 Bad Request**

**Content-Type: application/json; charset=utf-8**

****

[{"Message":"The Email field is not a valid e-mail
address.","Name":"Email"}]

  **\
**If we change the request message a little bit to expect xml as
response

****

**POST http://localhost:16913/api/Contact HTTP/1.1**

**Accept: text/xml**

**Content-Type: application/json**

**Host: localhost:16913**

**Content-Length: 31**

**{"fullname":"re","email":"foo"}**

We will get a list of errors formatted as xml this time

****

**HTTP/1.1 400 Bad Request**

**Content-Type: text/xml; charset=utf-8**

**\<?xml version="1.0" encoding="utf-8"?\>\<ArrayOfError
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xmlns:xsd="http://www.w3.org/2001/XMLSchema"\>\<Error\>\<Name\>Email\</Name\>\<Message\>The
Email field is not a valid e-mail
address.\</Message\>\</Error\>\</ArrayOfError\>**

