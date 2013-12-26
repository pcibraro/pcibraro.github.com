---
layout: post
title: "Implementing an identity provider and relying party with Zermatt and ASP.NET MVC"
date: 2008-09-01
comments: true
categories: ASP.NET
---

Zermatt is the framework recently released by Microsoft to develop
claim-aware applications. You can find some announcements
[here](http://blogs.msdn.com/vbertocci/archive/2008/07/09/announcing-the-beta-release-of-zermatt-developer-identity-framework.aspx)
and
[here](http://www.leastprivilege.com/TryQuotZermattquotAndGiveFeedback.aspx).

This framework supports the WS-Federation active and passive profiles.
This last one was initially designed with an unique purpose in mind,
allow the integration of "dumb clients" into the [identity
metasystem](http://en.wikipedia.org/wiki/Identity_Metasystem). As "dumb
clients", I am talking about clients like web browsers that do not have
the ability to handle cryptographic material.

All the magic is done through some consecutive Http redirects, and today
we will see how develop an identity provider and a relying party web
(with ASP.NET MVC) that are involved in the whole process.

The identity provider is based on the quickstart that is automatically
generated in Visual Studio when you create a new MVC web application.
This quickstart uses FormsAuthentication to authenticate the application
users and also provides an Account controller (that internally uses
ASP.NET Membership) to manage all those users. In order to integrate
Zermatt in this application, I added a new controller STSController that
knows to process messages for getting issue tokens with the user's
claims.

![](/images/legacy/Zermatt-MVC.gif)

For the relying party, Zermatt provides some web controls to
authenticate the user against the identity provider using the passive
profile. Unfortunately, for the simple fact that ASP.NET MVC does not
support controls with view state, we can not use them here. As
workaround, I created a couple of extensions methods that generate the
Urls for sending the corresponding messages to the identity provider
(Login and Logout).

public static class LoginUrlExtensions

{

   public static string LoginUrl(this UrlHelper helper, string
actionName, string controllerName, string stsUrl)

   {

       string host =
helper.ViewContext.HttpContext.Request.Url.Authority;

       string schema =
helper.ViewContext.HttpContext.Request.Url.Scheme;

 

       string realm = string.Format("{0}://{1}", schema, host);

       string reply = helper.Action(actionName,
controllerName).Substring(1);

 

       return
string.Format("{0}?wa=wsignin1.0&wtrealm={1}&wreply={2}&wctx=rm=0&id=FederatedPassiveSignIn1&wct={3}",

          stsUrl, realm, reply, XmlConvert.ToString(DateTime.Now));

   }

 

   public static string LogoutUrl(this UrlHelper helper, string
actionName, string controllerName, string stsUrl)

   {

       string host =
helper.ViewContext.HttpContext.Request.Url.Authority;

       string schema =
helper.ViewContext.HttpContext.Request.Url.Scheme;

 

       string realm = string.Format("{0}://{1}", schema, host);

       string reply = string.Format("{0}{1}", realm,
helper.Action(actionName, controllerName));

 

       return string.Format("{0}?wa=wsignout1.0&wreply={1}", stsUrl,
reply);

   }

}

The "actionName" and "controllerName" are just used to generate the
reply address where the user must be redirect after being authenticated
in the identity provider. These extension methods can be used in the
view as follow,

\<a href="\<%=Url.LoginUrl("Login", "Home",
"localhost://STS")%\>"\>Login\</a\>

We also need a method in the relying party to parse the RRST message and
generate a cookie with the user credentials and claims.

 

public interface IFederatedAuthentication

{

   IClaimsPrincipal Authenticate();

}

 

public class FederatedAuthentication : IFederatedAuthentication

{

     private string logoutUrl;

 

     public FederatedAuthentication(string logoutUrl)

     {

         this.logoutUrl = logoutUrl;

     }

 

     public IClaimsPrincipal Authenticate()

     {

         string securityTokenXml =
FederatedAuthenticationModule.Current.GetXmlTokenFromPassiveSignInResponse(System.Web.HttpContext.Current.Request,
null);

 

         FederatedAuthenticationModule current =
FederatedAuthenticationModule.Current;

 

         SecurityToken token = null;

         IClaimsPrincipal authContext =
current.AuthenticateUser(securityTokenXml, out token);

 

         TicketGenerationContext context = new
TicketGenerationContext(authContext, false, logoutUrl,
typeof(SignInControl).Name);

         current.IssueTicket(context);

 

         return authContext;

     }

}

 

As you can see in the code above, the Zermatt module
(FederatedAuthenticationModule) that parses the response message is tied
to the Request object, something that we do not have direct access from
a MVC controller (Well, it is bad practice if we want to test our code).
That's the reason I decided to put all that code in a pluggin that can
be injected later in the controller.

The complete solution is available to download from [this
location](/images/legacy/MVC.Zermatt.zip). Any feedback
would be great!!. Enjoy!!.

