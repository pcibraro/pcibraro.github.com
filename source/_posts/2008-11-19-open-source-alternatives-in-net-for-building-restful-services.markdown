---
layout: post
title: "Open Source alternatives in .NET for building RESTful services"
date: 2008-11-19
comments: true
categories: REST
---

Usually all my posts about REST are about WCF or mention this technology
in some parts. Today, I decided to take a different approach and discuss
some of the projects available today for building REST services, 
[Resourceful](http://www.codeplex.com/resourceful) and [Dream
framework](http://wiki.developer.mindtouch.com/Dream) (Both available
for mono as well).\
It is worth mentioning however that the WCF team has made an excellent
work introducing the new Web Model in .NET 3.5, it has definitively
helped a lot to adopt this kind of service in the .NET platform. In my
opinion, there are still some aspects in WCF that could be improved,

1.  WCF services are hard to unit test. It is possible but requires some
    extra work. I already mentioned some techniques  based on
    integration tests and mocks in this post ["Unit tests for
    WCF"](http://weblogs.asp.net/cibrax/archive/2008/05/16/unit-tests-for-wcf.aspx)
2.  Poor support for defining multiple resource representations/formats
    within a single operation definition.
3.  Any aspect you would like to add here ?

Ok, I will try now to summarize some of available features or
implementation details in these two projects.

**Resourceful**

-   Service definitions are totally imperative. Whereas a service
    definition (and operations) in WCF is made declaratively through
    attributes (annotating classes with WCF attributes), the service
    definition in Resourceful is totally imperative, it has to be made
    through several lines of code.

> LocalApplicationDescription app = new LocalApplicationDescription();
>
> // get-user
>
> LocalApplicationMethod getUser = app.NewMethod("getUser",
> HttpMethod.Get, \_usersController.GetUser);
>
> getUser.NewResponseRepresentation(MediaType.ApplicationXml);
>
> getUser.NewResponseRepresentation(MediaType.ApplicationExWwwFormUrlencoded);
>
>  
>
> ApplicationResource userResource = app.NewResource("users/{username}",
> new TemplateParameter("username", "xsd:string"));
>
> app.Bind(userResource, getUser);

> The developer has to perform two things, first define the operation
> itself specifying a friendly name along with the supported Http
> methods and resource representations and afterwards, create a resource
> mapping ("users/{username}" in this case). {username} is a URI
> template hole, equivalent to the Uri Templates in WCF.
>
> The method signature for the NewMethod is the following,
>
> NewMethod(string id, string name,Action\<IRepresentationContext\>
> handler)
>
> As you can see, the last argument is a delegate that points to the
> operation implementation. IRepresentationContext is equivalent to the
> WebOperationContext class in WCF, it contains all the runtime context
> settings that a service can use. This actually better than WCF because
> IRepresentationContext can be mocked for unit tests.
>
> In the example above, \_usersController is a simple class with the
> service implementation (This separation of concerns definitively helps
> a lot for unit testing). Some code for the GetUser operation
> implementation looks as follow,

> public void GetUser(IRepresentationContext context)
>
> {
>
>     string username = context.TemplateParameters["username"];
>
>  
>
>     UserAccount user = this.Engine.FindUser(username);
>
>  
>
>     if (user == null)
>
>     {
>
>         this.RenderStatus(context, HttpStatus.NotFound);
>
>         return;
>
>     }

>  

-   Support for WADL. The framework uses the service definition (the
    LocalApplicationDescription class in the example above) to publish
    all the available operations in the service.

>  ![](/images/legacy/wadl.jpg)

-   Support for multiple resource representations in a single operation.
    This is possible because the framework does not support the concept
    of channels (Or aspects) that can be plugged into the service to
    perform additional work, all the translation must be done in the
    operation implementation itself. The service operation can only get
    the resource representation as an stream from the
    IRepresentationContext (Same thing can be done in WCF if the input
    parameter for the operation method is an stream or a message),

> public void CreateUser(IRepresentationContext context)
>
> {
>
>     if (context.Request.MediaType != MediaType.ApplicationXml)
>
>     {
>
>         this.RenderStatus(context, HttpStatus.UnsupportedMediaType);
>
>         return;
>
>     }

     UserAccount user = UserAccount.FromXml(new
StreamReader(context.Request.GetEntityStream()));

> In the example above, CreateUser only supports POX (Plain old xml)
> representation for the users, so it also makes the translation from
> XML to an user entity.

-   There is not support for channels or aspects to perform additional
    work before a message arrives/leaves a service. There is, however,
    an special class for hosting service instances, it can be used from
    a console application or IIS as an http handler.

> Uri rootUri = new Uri(string.Format("http://localhost:3000/v1/",
> address));
>
>  
>
> BookmarkService service = new BookmarkService();
>
>  
>
> HttpServer host = new HttpServer(rootUri);
>
>  
>
> host.ReceiveWebRequest += delegate(HttpListenerContext context)
>
> {
>
>     service.Process(new HttpListenerContextAdapter(context, rootUri));
>
> };
>
>  
>
> host.Start();

-   Rich and fluent client API for consuming existing REST services.
    This is probably one of the nicest things about this framework, a
    lot of examples are provided for consuming well-know REST services
    on the web such as Amazon S3,  ADO.NET services, Simple DB or
    Delicious to name a few. This client API can be used from
    Silverlight applications as well.

**Dream Framework** 

-   Service definitions are declarative as in WCF. Two attributes are
    used, one for the service definition "DreamService" and another for
    each operation in that service, "DreamFeature".   

> [DreamService("Dream Tutorial Address-Book", "Copyright (c) 2006, 2007
> MindTouch, Inc.",
>
>       Info =
> "http://doc.opengarden.org/Dream\_SDK/Tutorials/Address\_Book",
>
>       SID = new string[] {
> "http://services.mindtouch.com/dream/tutorial/2007/03/addressbook" }
>
>     )]
>
>     public class AddressBookService : DreamService {
>
>         [DreamFeature("GET:firstname/{name}", "Get list of all
> addresses matching first name")]
>
>         public Yield GetFirstName(DreamContext context, DreamMessage
> request, Result\<DreamMessage\> response) {

>  
>
> I will not enter much in detail here, but the "DreamFeature" supports
> almost the same things as WCF and Resourceful, an Http verb and the
> URI template for the resource. For more information about how to
> create an Dream service from scratch, take a look at [this
> page](http://wiki.developer.mindtouch.com/Dream/Tutorials/Creating_a_Magic_8-Ball_service).

-   Every operation implementation should receive three arguments and
    return an enumerator. The arguments are basically the runtime
    context (Equivalent to WebOperationContext in WCF), the request
    message and a handler to send responses to the client.

> > using Yield = System.Collections.Generic.IEnumerator\<IYield\>;
> >
> > public Yield GetFirstName(DreamContext context, DreamMessage
> > request, Result\<DreamMessage\> response) {
>
> They implement a weird mechanism based on custom enumerators to
> execute callbacks on the service host. The service can basically
> returns IYield objects representing callbacks with the C\# keyword
> "yield". The response is automatically sent to the client application
> when the service invokes "yield break".

> [DreamFeature("GET:addresses", "Get all addresses")]
>
> public Yield GetAddresses(DreamContext context, DreamMessage request,
> Result\<DreamMessage\> response) {
>
>  
>
>     // send back the entire address book
>
>     lock(\_addresses) {
>
>         response.Return(DreamMessage.Ok(\_addresses));
>
>     }
>
>     yield break;
>
> }

-   Supports for long running services with durable state, it's not
    clear to me however how they restore that state between calls. The
    fields must be annotated with the attribute "DreamServiceState" in
    order to use this feature.

> [DreamServiceState]private XDoc \_addresses;

>  

