---
layout: post
title: "Running a partial SSL website with ASP.NET MVC"
date: 2009-01-19
comments: true
categories: ASP.NET
---

[Keith
Brown](http://www.pluralsight.com/community/blogs/keith/archive/2009/01/17/sslhelper-get-help-running-a-partial-ssl-website-in-asp-net.aspx)
has just released a helper class (Based on an [original
implementation](http://www.leastprivilege.com/CommentView.aspx?guid=50a39e64-caab-4fbd-b8e1-2be06b2e6081)
made by Dominick Baier) with very useful methods for mixing Http and
Https in a regular asp.net application. Before jumping in this post,
make sure to read his post (And Dominick's) to understand all the
problems you may have to deal with when implementing a partial SSL
website.

Running a partial SSL website in ASP.NET MVC is not much different,
however, in MVC, we can leverage the existing and powerful routing
mechanism to implement similar features.

**Switching to SSL through an ASP.NET Module**

This module  basically inspects the URL and does a redirect based on
some configuration where you specify routes that should be SSL
protected.

public void ProcessRequest(HttpContext context)

{

    if(Authentication.IsSslRequired() &&
context.Request.HttpMethod.Equals("get",
StringComparison.OrdinalIgnoreCase))

    {

        var data = RouteTable.Routes.GetRouteData(new
HttpContextWrapper(context));

        if (data != null)

        {

            if (!context.Request.IsSecureConnection)

            {

                if(data.DataTokens["isSecure"] != null &&
(bool)data.DataTokens["isSecure"])

                {

                    //Do redirect to https

                    var secureUrl =
context.Request.Url.ToString().ToSslUrl();

                    context.Response.Redirect(secureUrl, true);

                }

            }

            else

            {

                if (data.DataTokens["isSecure"] == null ||
!((bool)data.DataTokens["isSecure"]))

                {

                    //Do redirect to http

                    var unsecureUrl =
context.Request.Url.ToString().ToUnsecureUrl();

                    context.Response.Redirect(unsecureUrl, true);

                }

            }

        }

    }

}

As you can see in the code above, the module checks whether the page
should be ssl protected using a custom data token (IsSecure) that was
previously configured for the route, and afterwards, it redirects the
request according to that setting. For instance, if the route requires
to be the secure (IsSecure = true), and the request was actually made
through http, the module will redirect the request to https.

The absolute URLs are built using some extensions methods (ToSslUrl and
ToUnsecureUrl) that I will show later in this post.

Authentication.IsSslRequired is just an configuration setting that we
can use to skip all these checks during development.

public class Authentication

{

    public static bool IsSslRequired()

    {

        var setting = ConfigurationManager.AppSettings["RequireSSL"];

        if (setting != null)

        {

            return bool.Parse(setting);

        }

 

        return false;

    }

}

 

\<appSettings\>

  \<add key="RequireSSL" value="True"/\>

\</appSettings\>

The code for setting the custom data token in the MVC route looks as
follow,

routes.MapRoute(

    "MyAccount/PasswordChange",

    "MyAccount/PasswordChange",

    new { controller = "Accounts", action = "PasswordChange" }

).DataTokens = new RouteValueDictionary(new { isSecure = true });

**** 

**Extension methods for generating absolute URLs**

As I showed before in the ASP.NET Module, two extensions methods were
used to generate absolute URLs. I just based their implementation on
some code originally written by Troy Goode in this
[post](http://www.squaredroot.com/post/2008/06/MVC-and-SSL.aspx). \

/// \<summary\>

/// Provides helper extensions for turning strings into fully-qualified
and SSL-enabled Urls.

/// \</summary\>

public static class UrlStringExtensions

{

    /// \<summary\>

    /// Takes a relative or absolute url and returns the fully-qualified
url path.

    /// \</summary\>

    /// \<param name="text"\>The url to make fully-qualified. Ex:
Home/About\</param\>

    /// \<returns\>The absolute url plus protocol, server, & port. Ex:
http://localhost:1234/Home/About\</returns\>

    public static string ToFullyQualifiedUrl( this string text )

    {

        var oldUrl = text;

        var oldUrlArray = ( oldUrl.Contains( "?" ) ? oldUrl.Split( '?' )
: new[]{ oldUrl, "" } );

 

        var requestUri = HttpContext.Current.Request.Url;

        var localPathAndQuery = requestUri.LocalPath + requestUri.Query;

        var urlBase = requestUri.AbsoluteUri.Substring( 0,
requestUri.AbsoluteUri.Length - localPathAndQuery.Length );

 

        var newUrl = VirtualPathUtility.ToAbsolute( oldUrlArray[0] );

        if( !string.IsNullOrEmpty( oldUrlArray[1] ) )

            newUrl += "?" + oldUrlArray[1];

 

        return urlBase + newUrl;

    }

 

    /// \<summary\>

    /// Looks for Html links in the passed string and turns each
relative or absolute url and returns the fully-qualified url path.

    /// \</summary\>

    /// \<param name="text"\>The url to make fully-qualified. Ex: \<a
href="Home/About"\>Blah\</a\>\</param\>

    /// \<returns\>The absolute url plus protocol, server, & port. Ex:
\<a href="http://localhost:1234/Home/About"\>Blah\</a\>\</returns\>

    public static string ToFullyQualifiedLink( this string text )

    {

        var regex = new Regex(

           
"(?\<Before\>\<a.\*href=\\")(?!http)(?\<Url\>.\*?)(?\<After\>\\".+\>)",

            RegexOptions.Multiline | RegexOptions.IgnoreCase

            );

 

        return regex.Replace( text, ( Match m ) =\>

                                    m.Groups["Before"].Value +

                                    ToFullyQualifiedUrl(
m.Groups["Url"].Value ) +

                                    m.Groups["After"].Value

            );

    }

 

    /// \<summary\>

    /// Takes a relative or absolute url and returns the fully-qualified
url path using the Https protocol.

    /// \</summary\>

    /// \<param name="text"\>The url to make fully-qualified. Ex:
Home/About\</param\>

    /// \<returns\>The absolute url plus server, & port using the Https
protocol. Ex: https://localhost:1234/Home/About\</returns\>

    public static string ToSslUrl( this string text )

    {

        if (IsSslRequired())

        {

            string absoluteUrl = null;

            if (text.StartsWith("http://",
StringComparison.OrdinalIgnoreCase) || text.StartsWith("https://",
StringComparison.OrdinalIgnoreCase))

                absoluteUrl = text;

            else

                absoluteUrl = ToFullyQualifiedUrl(text);

 

            return absoluteUrl.Replace("http:", "https:");

        }

        else

        {

            return text;

        }

    }

 

    /// \<summary\>

    /// Takes a relative or absolute url and returns the fully-qualified
url path using the Http protocol.

    /// \</summary\>

    /// \<param name="text"\>The url to make fully-qualified. Ex:
Home/About\</param\>

    /// \<returns\>The absolute url plus server, & port using the Http
protocol. Ex: http://localhost:1234/Home/About\</returns\>

    public static string ToUnsecureUrl(this string text)

    {

        string absoluteUrl = null;

        if (text.StartsWith("http://",
StringComparison.OrdinalIgnoreCase) || text.StartsWith("https://",
StringComparison.OrdinalIgnoreCase))

            absoluteUrl = text;

        else

            absoluteUrl = ToFullyQualifiedUrl(text);

 

        return absoluteUrl.Replace("https:", "http:");

    }

 

    /// \<summary\>

    /// Looks for Html links in the passed string and turns each
relative or absolute url into a fully-qualified url path using the Https
protocol.

    /// \</summary\>

    /// \<param name="text"\>The url to make fully-qualified. Ex: \<a
href="Home/About"\>Blah\</a\>\</param\>

    /// \<returns\>The absolute url plus server, & port using the Https
protocol. Ex: \<a
href="https://localhost:1234/Home/About"\>Blah\</a\>\</returns\>

    public static string ToSslLink( this string text )

    {

        if (IsSslRequired())

        {

            return ToFullyQualifiedLink(text).Replace("http:",
"https:");

        }

        else

        {

            return ToFullyQualifiedLink(text);

        }

    }

 

    private static bool IsSslRequired()

    {

        return Authentication.IsSslRequired();

    }

}

\
They can be used from a view as well to generate links with complete
Urls,

\<a href='\<% =Url.Action("PasswordChange", "Accounts").ToSslUrl()
%\>'\>Change your password\</a\>

**Gettting an absolute URL within a controller**

For some scenarios, we might need to redirect the user from a secure
route to a regular route or viceversa within a controller. In those
cases, we should have a way to generate a complete URL from a controller
method and used it later with a RedirectResult (This is also useful when
you want to include a link to a web page in an email sent by your
website).

For instance, it would be very helpful to have something like this,

return new RedirectResult(this.FullActionUrl\<MyController\>(c =\>
c.SecureMethod(), "https"));

Where "FullActionUrl" is an extension method for the Controller class.
In addition to the controller method we want to use to get the absolute
route ("SecureMethod"), we can also specify the scheme to be used with
that route.

The code for doing that is shown bellow (They are part of the .NETfx
project,
[http://code.google.com/p/netfx/](http://code.google.com/p/netfx/))

/// \<summary\>

/// Returns the full URL for performing an invocation to an action based

/// on an expression representing an invocation to a controller method

/// that may include arguments.

/// \</summary\>

/// \<typeparam name="T"\>Type of the controller to call to. Can be
omitted as it can be inferred from the action type.\</typeparam\>

/// \<param name="controller"\>The controller performing the
call.\</param\>

/// \<param name="action"\>The action containing the call.\</param\>

public static string FullActionUrl\<T\>(this Controller controller,
Expression\<Action\<T\>\> action, string scheme)

    where T : Controller

{

    string host = controller.HttpContext.Request.Url.Authority;

    string virtualPath = ActionUrl(controller, action);

 

    return string.Format("{0}://{1}{2}", scheme, host, virtualPath);

}

 

/// \<summary\>

/// Returns the URL for performing an invokation to an action based

/// on an expression representing an invocation to a controller method

/// that may include arguments.

/// \</summary\>

/// \<typeparam name="T"\>Type of the controller to call to. Can be
omitted as it can be inferred from the action type.\</typeparam\>

/// \<param name="controller"\>The controller performing the
call.\</param\>

/// \<param name="action"\>The action containing the call.\</param\>

public static string ActionUrl\<T\>(this ControllerBase controller,
Expression\<Action\<T\>\> action)

    where T : Controller

{

    var call = ControllerExpression.GetMethodCall\<T\>(action);

 

    string actionName = call.Method.Name;

    string controllerName =
ControllerExpression.GetControllerName\<T\>();

 

    var values = LinkBuilder.BuildParameterValuesFromExpression(call);

    values.Add("action", actionName);

    values.Add("controller", controllerName);

 

    var vpd =
RouteTable.Routes.GetVirtualPath(controller.ControllerContext, values);

    string target = null;

    if (vpd != null)

    {

        target = vpd.VirtualPath;

    }

 

    return target;

}

The complete code is available to download from
[here](/images/legacy/MVC.SSL.zip). 

