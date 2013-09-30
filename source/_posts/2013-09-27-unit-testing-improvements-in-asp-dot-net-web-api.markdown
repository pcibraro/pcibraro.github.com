---
layout: post
title: "Unit Testing Improvements in ASP.NET Web API"
date: 2013-09-27 12:46
comments: true
categories: webapi
---

A time ago I discussed some serious issues in ASP.NET Web API for testing controllers that were using the Request or UrlHelper instances because those required a valid HttpConfiguration instance. You could initialize the HttpConfiguration instace, but that required a lot of work and some ugly code as part of your tests.

The following method in a controller works to describe the problem,

```csharp
public HttpResponseMessage Post(User model)
{
  var response = Request.CreateResponse(HttpStatusCode.Created, user);
  var link = Url.Link("DefaultApi", new { id = id, controller = "Users" });
  response.Headers.Location = new Uri(link);
  return response;
}
```

The Request propery is being used to generate a content negotiated response, and the Url property to infer the new location for the resource. If you use ASP.NET Web API as it is today, you will have to write a lot of custom code to initialize the configuration instance as it is shown below.

```csharp
var controller = new UserController();
controller.Configuration = new HttpConfiguration();
var route = controller.Configuration.Routes.MapHttpRoute(
         name: "DefaultApi",
         routeTemplate: "api/{controller}/{id}",
         defaults: new { id = RouteParameter.Optional }
);
var routeData = new HttpRouteData(route, 
       new HttpRouteValueDictionary 
       { 
          { "id", "1" },
          { "controller", "Users" } 
       }
);
controller.Request = new HttpRequestMessage(HttpMethod.Post, 
  "http://localhost:9091/");
controller.Request.Properties
  .Add(HttpPropertyKeys.HttpConfigurationKey, controller.Configuration);
controller.Request.Properties
  .Add(HttpPropertyKeys.HttpRouteDataKey, routeData);
```

Glenn Block wrote a very nice extension as part of the Web API Contrib project for making this configuration much more simpler. This extension is available here, https://github.com/WebApiContrib/WebAPIContrib/blob/master/src/WebApiContrib.Testing/ApiControllerExtensions.cs. 

However, we can left all this in the past with the new ASP.NET Web API vNext release. The introduction of the IHttpActionResult interface (Equivalent to ActionResult in ASP.NET MVC) has simplified a lot the unit testing story for controllers. A controller method can return now an implementation of IHttpActionResult, which internally uses the Request or the UrlHelper for link generation, so the unit test only cares about the returned IHttpActionResult instance.

The following code shows the same controller method using an instance of IHttpActionResult.

```csharp
public CreatedAtRouteNegotiatedContentResult<UserModel> Post(UserModel user)
{
  user.Id = 1;

  var result = new CreatedAtRouteNegotiatedContentResult<UserModel>(
    "DefaultApi",
	  new Dictionary<string, object> { { "id", user.Id } },
      user,
      this);

  return result;
}
```

CreatedAtRouteNegotiatedContentResult is an implementation also included in the framework for handling this scenario. A new resource is created and the location is set in the response message. 

The code for the unit test is much simpler too.

```csharp
[TestMethod]
public void ShouldCreateUser()
{
    var controller = new UserController();
    var result = controller.Post(new UserModel { Name = "foo" });

    Assert.IsInstanceOfType(result.Content, typeof(UserModel));
    Assert.IsNotNull(result);
}
```

The UrlHelper class has also been modified to make most of its methods virtual, so they can be mocked or overriden as part of an unit test. You can use this in case you still need to rely on the Url instance in the controller. For example, if you have a controller method like this one.

```csharp
public HttpResponseMessage Post(UserModel user)
{
  user.Id = 1;

  var response = new HttpResponseMessage(HttpStatusCode.Created);
  var link = Url.Link("DefaultApi", 
    new { id = user.Id, controller = "User" });

  response.Headers.Location = new Uri(link);

  return response;
}
```

You can create a custom class that derives from UrlHelper and overrides the Link method.

```csharp
[TestMethod]
public void ShouldCreateUser()
{
  var controller = new UserController();
  controller.Url = new MyUrlHelper();

  var result = controller.Post(
    new UserModel { Name = "foo" });

  Assert.IsNotNull(result);
}

public class MyUrlHelper : UrlHelper
{
  public override string Link(string routeName, object routeValues)
  {
    return "http://example.com/user/1";
  }
}
```







