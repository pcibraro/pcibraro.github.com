---
layout: post
title: "Unit testing ASP.NET Web API controllers that rely on the UrlHelper"
date: 2012-10-10
comments: true
categories: .NET
---

UrlHelper is the class you can use in ASP.NET Web API to automatically
infer links from the routing table without hardcoding anything. For
example, the following code uses the helper to infer the location url
for a new resource,

    public HttpResponseMessage Post(User model)

~~~~ {style="font-size: 12px; font-family: consolas,'Courier New',courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"}
{
~~~~

~~~~ {style="font-size: 12px; font-family: consolas,'Courier New',courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"}
  var response = Request.CreateResponse(HttpStatusCode.Created, user);
~~~~

~~~~ {style="font-size: 12px; font-family: consolas,'Courier New',courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"}
  var link = Url.Link("DefaultApi", new { id = id, controller = "Users" });
~~~~

~~~~ {style="font-size: 12px; font-family: consolas,'Courier New',courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"}
  response.Headers.Location = new Uri(link);
~~~~

~~~~ {style="font-size: 12px; font-family: consolas,'Courier New',courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"}
  return response;
~~~~

~~~~ {style="font-size: 12px; font-family: consolas,'Courier New',courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"}
}
~~~~

That code uses a previously defined route “DefaultApi”, which you might
configure in the HttpConfiguration object (This is the route generated
by default when you create a new Web API project).

The problem with UrlHelper is that it requires from some initialization
code before you can invoking it from a unit test (for testing the Post
method in this example). If you don’t initialize the HttpConfiguration
and Request instances associated to the controller from the unit test,
it will fail miserably.

After digging into the ASP.NET Web API source code a little bit, I could
figure out what the requirements for using the UrlHelper are. It relies
on the routing table configuration, and a few properties you need to add
to the HttpRequestMessage. The following code illustrates what’s needed,

    var controller = new UserController();

~~~~ {style="font-size: 12px; font-family: consolas,'Courier New',courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"}
controller.Configuration = new HttpConfiguration();
~~~~

~~~~ {style="font-size: 12px; font-family: consolas,'Courier New',courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"}
var route = controller.Configuration.Routes.MapHttpRoute(
~~~~

~~~~ {style="font-size: 12px; font-family: consolas,'Courier New',courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"}
         name: "DefaultApi",
~~~~

~~~~ {style="font-size: 12px; font-family: consolas,'Courier New',courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"}
         routeTemplate: "api/{controller}/{id}",
~~~~

~~~~ {style="font-size: 12px; font-family: consolas,'Courier New',courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"}
         defaults: new { id = RouteParameter.Optional }
~~~~

~~~~ {style="font-size: 12px; font-family: consolas,'Courier New',courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"}
);
~~~~

~~~~ {style="font-size: 12px; font-family: consolas,'Courier New',courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"}
~~~~

~~~~ {style="font-size: 12px; font-family: consolas,'Courier New',courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"}
var routeData = new HttpRouteData(route, 
~~~~

~~~~ {style="font-size: 12px; font-family: consolas,'Courier New',courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"}
       new HttpRouteValueDictionary 
~~~~

~~~~ {style="font-size: 12px; font-family: consolas,'Courier New',courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"}
       { 
~~~~

~~~~ {style="font-size: 12px; font-family: consolas,'Courier New',courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"}
          { "id", "1" },
~~~~

~~~~ {style="font-size: 12px; font-family: consolas,'Courier New',courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"}
          { "controller", "Users" } 
~~~~

~~~~ {style="font-size: 12px; font-family: consolas,'Courier New',courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"}
       }
~~~~

~~~~ {style="font-size: 12px; font-family: consolas,'Courier New',courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"}
);
~~~~

~~~~ {style="font-size: 12px; font-family: consolas,'Courier New',courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"}
~~~~

~~~~ {style="font-size: 12px; font-family: consolas,'Courier New',courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"}
controller.Request = new HttpRequestMessage(HttpMethod.Post, "http://localhost:9091/");
~~~~

~~~~ {style="font-size: 12px; font-family: consolas,'Courier New',courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"}
controller.Request.Properties.Add(HttpPropertyKeys.HttpConfigurationKey, controller.Configuration);
~~~~

~~~~ {style="font-size: 12px; font-family: consolas,'Courier New',courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"}
controller.Request.Properties.Add(HttpPropertyKeys.HttpRouteDataKey, routeData);
~~~~

     

The HttpRouteData instance should be initialized with the route values
you will use in the controller method (“id” and “controller” in this
example). Once you have correctly setup all those properties, you
shouldn’t have any problem to use the UrlHelper. There is no need to
mock anything else.

Enjoy!!.

