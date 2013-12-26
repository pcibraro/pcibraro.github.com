---
layout: post
title: "Using the WCF OAuth channel with an ADO.NET service"
date: 2008-11-14
comments: true
categories: Entity-Framework
---

As I promised in my previous post "OAuth Channel for WCF RESTful
services", it is now time to show this new channel in action with a real
service. To make this sample more interesting, I decided to base this
implementation on an ADO.NET service that provides information about
contacts.

This post will be a kind of walk-through to demonstrate all the steps
required to implement the ADO.NET service, and then, the final
integration with OAuth.

**1.Create a custom data source (IQueryable) implementation for using
with the ADO.NET data service**

[DataServiceKey("Id")]

public class Contact

{

    public int Id { get; set; }

    public string Name { get; set; }

    public string Email { get; set; }

    public string Owner { get; set; }

}

 

public class ContactsData

{

    static Contact[] \_contacts;

 

    static ContactsData()

    {

        \_contacts = new Contact[]{

          new Contact(){ Id=0, Name="Mike", Email="mike@contoso.com",
Owner = "jane" },

          new Contact(){ Id=1, Name="Saaid", Email="Saaid@hotmail.com",
Owner = "jane"},

          new Contact(){ Id=2, Name="John", Email="j123@live.com", Owner
= "john"},

          new Contact(){ Id=3, Name="Pablo", Email="Pablo@mail.com",
Owner = "john"}};

    }

 

    public IQueryable\<Contact\> Contacts

    {

        get { return \_contacts.AsQueryable\<Contact\>(); }

    }

 

}

What I defined here is a simple Contact class representing the contact
entity and a ContactsData for the ADO.NET service data source. The
service will automatically reflect the IQueryable properties in this
data source class. The "DataServiceKey" attribute on top of the contact
entity is required by ADO.NET services to define an artificial primary
key on custom classes (It took me some to figure this out).

**2. Implement the ADO.NET data service**

public class contacts : DataService\<ContactsData\>

{

    // This method is called only once to initialize service-wide
policies.

    public static void InitializeService(IDataServiceConfiguration
config)

    {

        config.SetEntitySetAccessRule("\*", EntitySetRights.AllRead);

    }

 

    [QueryInterceptor("Contacts")]

    public Expression\<Func\<Contact, bool\>\> OnQueryContact()

    {

        var name = Thread.CurrentPrincipal.Identity.Name;

        return c =\> c.Owner.Equals(name,
StringComparison.OrdinalIgnoreCase);

    }

}

The "QueryInterceptor" in this service implementation basically filters
the resulting contacts based on the authenticated user. As I showed in
my previous post, the authentication is performed by the OAuth channel.

**3. Configure the OAuth WCF channel for the ADO.NET data service**

\<%@ ServiceHost Language="C\#"
Factory="ExampleOAuthChannel.AppServiceHostFactory"
Service="ADOServices.OAuth.contacts" %\>

using System;\
using System.ServiceModel;\
using System.ServiceModel.Activation;\
using Microsoft.ServiceModel.Web;

namespace ExampleOAuthChannel \
{\
  class AppServiceHostFactory : ServiceHostFactory\
  {\
    protected override ServiceHost CreateServiceHost(Type serviceType,
Uri[] baseAddresses)\
    {\
        WebServiceHost2 result = new WebServiceHost2(serviceType, true,
baseAddresses);\
        result.Interceptors.Add(new OAuthChannel.OAuthInterceptor(\
   ADOServices.OAuth.OAuthServicesLocator.Provider,
ADOServices.OAuth.OAuthServicesLocator.AccessTokenRepository));\
        return result;\
    }\
  }\
}

Nothing new in this step, I only registered the OAuth interceptor in the
WCF service host for the ADO.NET service.

[Download the complete
example](/images/legacy/ADOServices_OAuth.zip). (It
includes a client application implementation as well)

