---
layout: post
title: "Making ajax calls with Hawk authentication to ASP.NET Web API"
date: 2014-01-31 14:14
comments: true
categories: webapi
---

Hawk is an authentication protocol initially written by Eran Hammer in javascript for being used in Node.js. Later on, Eran also added support for browsers through a browser.js library, which is also part of the [hawk github project](https://github.com/hueniverse/hawk). 

As part of this post, I will show how you can use that browser.js library to make Ajax calls to a ASP.NET Web API that authenticates the calls with Hawk.

The first thing is to define the Web API to call from javascript. For the sake of simplicity, we will use a very simple "Hello World" controller that returns the name of the authenticated user.

```csharp
public class HelloWorldController : ApiController
{
  public string Get()
  {
    return "hello " + this.User.Identity.Name;
  }
}
```

As second step, we will configure the Hawk filter globally to authenticate the calls. This filter is part of the HawkNet integration project with Web API. 

```csharp
public static void Register(HttpConfiguration config)
{
  config.Filters.Add(new RequiresHawkAttribute((id) =>
  {
    return new HawkCredential
    {
      Id = "dh37fgj492je",
      Key = "werxhqb98rpaxn39848xrunpaw3489ruxnpa98w4rxn",
      Algorithm = "hmacsha256",
      User = "steve"
    };
  }));
} 
```

Once we have the Web API configured, we will write the javascript to make the call, which will use browser.js to generate the hawk header on the client. 

```html
<script src="~/Scripts/browser.js"></script>
<script>
$(function () {
    var url = 'http://localhost:28290/api/HelloWorld';

	var credentials = {
        id: 'dh37fgj492je', // Required by Hawk.client.header
        key: 'werxhqb98rpaxn39848xrunpaw3489ruxnpa98w4rxn',
        algorithm: 'sha256',
        user: 'Steve'
    };

	$("#helloworld").click(function() {
				
	    var header = hawk.client.header(url, 'GET', { credentials: credentials });

	    $.ajax({
	        dataType: "json",
	        headers: { authorization: header.field },
	        url: url,
	        success: function (error, response, body) {
	            
	            $("#response").html(body.responseText);
	        }
	    });
	});
});
</script>
```

As you can see, the part that matters in the sample above is how you generate the hawk header by calling the "header" method in the client object provided by the browser.js library. That method receives the url, the http method and the credentials. The result of that call is the hawk authentication header, which is attached to the authorization header in the ajax call. 

The complete sample can be found in the [HawkNet github project](https://github.com/pcibraro/hawknet/tree/master/Example.Web)