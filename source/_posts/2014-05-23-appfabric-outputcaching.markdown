---
layout: post
title: "AppFabric OutputCaching"
date: 2014-05-23 14:36
comments: true
categories: WebAPI
---

ASP.NET Web API does not provide any output caching capabilities out of the box other than the ones you would traditionally find in the ASP.NET caching module. 
Fortunately, Filip [wrote a very nice library](http://www.strathweb.com/2012/05/output-caching-in-asp-net-web-api/) that you can use to decorate your Web API controller methods with an [OutputCaching] attribute, which is similar to the one you can find in ASP.NET MVC. 
This library provides a way to configure different persistence storages for the cached data, which uses memory by default. As part of this post, I will show how you can implement your own persistence provider for AppFabric in order to support distributed caching on web applications running on premises. 

The first thing is to install AppFabric Caching. Wade Wegner wrote a very useful post describing all the required steps [here](http://www.wadewegner.com/2010/08/getting-started-with-windows-server-appfabric-cache/).

Once AppFabric Cache is installed, you need to start the cluster and configure a new cache that will be used by our extension. Open up the Cache PowerShell console (Caching Administration Windows PowerShell in programs). Run the following command in the PowerShell console:

Start-CacheCluster

Create a new cache for our extension. Run the following command in the PowerShell console:

New-Cache OutputCache

At that point, we are ready to jump into the implementation of the extension. Before writing any code, we will use the AppFabric Caching client library, which is available as a Nuget package. You can find it in the repository under the name of "ServerAppFabric.Client". 
The library written by Filip provides an extension point for the persistence providers called IApiOutputCache, so we will have to implement that interface.

```csharp
public class AppFabricCachingProvider : IApiOutputCache
{
    readonly static DataCacheFactory Factory = new DataCacheFactory();
    const string Region = "OutputCache";
    readonly DataCache cache;

    public AppFabricCachingProvider(string cacheName)
    {
        this.cache = Factory.GetCache(cacheName);
        this.cache.CreateRegion(Region);
    }

    public void Add(string key, object o, DateTimeOffset expiration, string dependsOnKey = null)
    {
        var exp = expiration - DateTime.Now;

        if (dependsOnKey == null)
        {
            dependsOnKey = key;
        }

        cache.Put(key, o, new[] { new DataCacheTag(dependsOnKey) }, Region);
    }

    public bool Contains(string key)
    {
        var result = this.cache.Get(key, Region);
        if (result != null) return true;

        return false;
    }

    public object Get(string key)
    {
        var result = this.cache.Get(key, Region);
        return result; 
    }

    public T Get<T>(string key) where T : class
    {
        var result = this.cache.Get(key, Region) as T;
        return result; 
    }

    public void Remove(string key)
    {
        this.cache.Remove(key, Region);
    }

    public void RemoveStartsWith(string key)
    {
        var objs = this.cache.GetObjectsByTag(new DataCacheTag(key), Region);
        foreach(var o in objs)
        {
            this.cache.Remove(o.Key, Region);
        }
    }
}
```

This implementation is pretty straightforward and uses the AppFabric client library for getting or storing data in the cache. All the data is stored as part of a region, which groups all the entries together to facilitate management. The name of the cache is passed as part of the constructor, so it will be provided at the moment of instantiating and configuring this extension.

The following code shows how the extension is configured,

```csharp
var config = new HttpSelfHostConfiguration("http://localhost:999");
config.Routes.MapHttpRoute(
      name: "DefaultApi",
    routeTemplate: "api/{controller}/{id}",
    defaults: new { id = RouteParameter.Optional }
);
var server = new HttpSelfHostServer(config);

config.CacheOutputConfiguration().RegisterCacheOutputProvider(() => new AppFabricCachingProvider("OutputCache"));

server.OpenAsync().Wait();

Console.ReadKey();

server.CloseAsync().Wait();
```

The name of the cache must be the same that we used before in the powershell console with the New-Cache command.  

Finally, output caching can be configured in any existing controller using the [OutputCaching] attribute distributed with the Filip's library.

```csharp
public class TeamsController : ApiController
{

    [CacheOutput(ClientTimeSpan = 50, ServerTimeSpan = 50)]
    public IEnumerable<Team> Get()
    {
        return Teams;
    }
```

The implementation of this provider is available as a [separate project in GitHub](https://github.com/pcibraro/CacheOutput.AppFabric).

