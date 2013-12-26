---
layout: post
title: "Authenticating clients in the new WCF Http stack"
date: 2011-02-04
comments: true
categories: .NET
---

About this time last year, I wrote a couple of posts about how to use
the “Interceptors” from the REST starker kit for implementing several
authentication mechanisms like “SAML”, “Basic Authentication” or “OAuth”
in the WCF Web programming model. The things have changed a lot since
then, and [Glenn](http://blogs.msdn.com/b/gblock/) finally put on our
hands a new version of the [Web programming
model](http://wcf.codeplex.com/) that deserves some attention and I
believe will help us a lot to build more Http oriented services in the
.NET stack. What you can get today from wcf.codeplex.com is a preview
with some cool features like Http Processors (which I already discussed
[here](http://weblogs.asp.net/cibrax/archive/2010/11/15/http-processors-in-the-wcf-web-programming-model.aspx)),
a new and improved version of the HttpClient library, Dependency
injection and better TDD support among others.

However, the framework still does not support an standard way of doing
client authentication on the services (This is something planned for the
upcoming releases I believe). For that reason, moving the existing
authentication interceptors to this new programming model was one of the
things I did in the last few days.

In order to make authentication simple and easy to extend,  I first came
up with a model based on what I called “Authentication Interceptors”. An
authentication interceptor maps to an existing Http authentication
mechanism and implements the following interface,

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: white; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
public interface IAuthenticationInterceptor
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: #f4f4f4; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
{
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: white; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
    string Scheme { get; }
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: #f4f4f4; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
    bool DoAuthentication(HttpRequestMessage request, HttpResponseMessage response, out IPrincipal principal);
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: white; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
}
~~~~

An authentication interceptors basically needs to returns the http
authentication schema that implements in the property “Scheme”, and
implements the authentication mechanism in the method
“DoAuthentication”. As you can see, this last method “DoAuthentication”
only relies on the HttpRequestMessage and HttpResponseMessage classes,
making the testing of this interceptor very simple (There is no need to
do some black magic with the WCF context or messages).

After this, I implemented a couple of interceptors for supporting basic
authentication and brokered authentication with SAML (using WIF) in my
services. The following code illustrates how the basic authentication
interceptors looks like.

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: white; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
public class BasicAuthenticationInterceptor : IAuthenticationInterceptor
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: #f4f4f4; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
{
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: white; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
    Func<UsernameAndPassword, bool> userValidation;
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: #f4f4f4; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
    string realm;
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: white; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
 
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: #f4f4f4; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
    public BasicAuthenticationInterceptor(Func<UsernameAndPassword, bool> userValidation, string realm)
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: white; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
    {
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: #f4f4f4; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
        if (userValidation == null)
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: white; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
            throw new ArgumentNullException("userValidation");
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: #f4f4f4; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
 
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: white; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
        if (string.IsNullOrEmpty(realm))
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: #f4f4f4; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
            throw new ArgumentNullException("realm");
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: white; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
 
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: #f4f4f4; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
        this.userValidation = userValidation;
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: white; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
        this.realm = realm;
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: #f4f4f4; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
    }
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: white; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
 
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: #f4f4f4; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
    public string Scheme
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: white; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
    {
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: #f4f4f4; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
        get { return "Basic"; }
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: white; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
    }
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: #f4f4f4; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
 
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: white; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
    public bool DoAuthentication(HttpRequestMessage request, HttpResponseMessage response, out IPrincipal principal)
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: #f4f4f4; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
    {
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: white; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
        string[] credentials = ExtractCredentials(request);
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: #f4f4f4; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
        if (credentials.Length == 0 || !AuthenticateUser(credentials[0], credentials[1]))
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: white; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
        {
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: #f4f4f4; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
            response.StatusCode = HttpStatusCode.Unauthorized;
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: white; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
            response.Content = new StringContent("Access denied");
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: #f4f4f4; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
            response.Headers.WwwAuthenticate.Add(new AuthenticationHeaderValue("Basic", "realm=" + this.realm));
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: white; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
 
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: #f4f4f4; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
            principal = null;
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: white; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
 
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: #f4f4f4; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
            return false;
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: white; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
        }
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: #f4f4f4; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
        else
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: white; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
        {
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: #f4f4f4; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
            principal = new GenericPrincipal(new GenericIdentity(credentials[0]), new string[] {});
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: white; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
 
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: #f4f4f4; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
            return true;
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: white; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
        }
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: #f4f4f4; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
    }
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: white; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
 
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: #f4f4f4; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
    private string[] ExtractCredentials(HttpRequestMessage request)
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: white; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
    {
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: #f4f4f4; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
        if (request.Headers.Authorization != null && request.Headers.Authorization.Scheme.StartsWith("Basic"))
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: white; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
        {
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: #f4f4f4; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
            string encodedUserPass = request.Headers.Authorization.Parameter.Trim();
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: white; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
 
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: #f4f4f4; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
            Encoding encoding = Encoding.GetEncoding("iso-8859-1");
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: white; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
            string userPass = encoding.GetString(Convert.FromBase64String(encodedUserPass));
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: #f4f4f4; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
            int separator = userPass.IndexOf(':');
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: white; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
 
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: #f4f4f4; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
            string[] credentials = new string[2];
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: white; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
            credentials[0] = userPass.Substring(0, separator);
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: #f4f4f4; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
            credentials[1] = userPass.Substring(separator + 1);
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: white; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
 
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: #f4f4f4; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
            return credentials;
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: white; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
        }
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: #f4f4f4; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
 
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: white; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
        return new string[] { };
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: #f4f4f4; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
    }
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: white; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
 
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: #f4f4f4; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
    private bool AuthenticateUser(string username, string password)
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: white; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
    {
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: #f4f4f4; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
        var usernameAndPassword = new UsernameAndPassword
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: white; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
        {
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: #f4f4f4; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
            Username = username,
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: white; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
            Password = password
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: #f4f4f4; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
        };
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: white; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
 
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: #f4f4f4; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
        if (this.userValidation(usernameAndPassword))
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: white; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
        {
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: #f4f4f4; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
            return true;
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: white; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
        }
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: #f4f4f4; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
 
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: white; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
        return false;
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: #f4f4f4; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
    }
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: white; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
}
~~~~

This interceptor receives in the constructor a callback in the form of a
Func delegate for authenticating the user and the “realm”, which is
required as part of the implementation. The rest is a general
implementation of the basic authentication mechanism using standard http
request and response messages.

I also implemented another interceptor for authenticating a SAML token
with WIF.

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: white; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
public class SamlAuthenticationInterceptor : IAuthenticationInterceptor
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: #f4f4f4; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
{
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: white; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
    SecurityTokenHandlerCollection handlers = null;
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: #f4f4f4; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
 
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: white; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
    public SamlAuthenticationInterceptor(SecurityTokenHandlerCollection handlers)
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: #f4f4f4; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
    {
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: white; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
        if (handlers == null)
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: #f4f4f4; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
            throw new ArgumentNullException("handlers");
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: white; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
 
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: #f4f4f4; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
        this.handlers = handlers;
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: white; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
    }
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: #f4f4f4; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
 
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: white; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
    public string Scheme
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: #f4f4f4; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
    {
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: white; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
        get { return "saml"; }
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: #f4f4f4; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
    }
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: white; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
 
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: #f4f4f4; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
    public bool DoAuthentication(HttpRequestMessage request, HttpResponseMessage response, out IPrincipal principal)
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: white; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
    {
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: #f4f4f4; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
        SecurityToken token = ExtractCredentials(request);
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: white; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
 
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: #f4f4f4; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
        if (token != null)
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: white; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
        {
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: #f4f4f4; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
            ClaimsIdentityCollection claims = handlers.ValidateToken(token);
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: white; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
 
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: #f4f4f4; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
            principal = new ClaimsPrincipal(claims);
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: white; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
 
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: #f4f4f4; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
            return true;
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: white; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
        }
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: #f4f4f4; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
        else
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: white; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
        {
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: #f4f4f4; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
            response.StatusCode = HttpStatusCode.Unauthorized;
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: white; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
            response.Content = new StringContent("Access denied");
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: #f4f4f4; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
 
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: white; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
            principal = null;
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: #f4f4f4; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
 
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: white; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
            return false;
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: #f4f4f4; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
        }
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: white; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
    }
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: #f4f4f4; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
 
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: white; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
    private SecurityToken ExtractCredentials(HttpRequestMessage request)
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: #f4f4f4; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
    {
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: white; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
        if (request.Headers.Authorization != null && request.Headers.Authorization.Scheme == "saml")
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: #f4f4f4; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
        {
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: white; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
            XmlTextReader xmlReader = new XmlTextReader(new StringReader(request.Headers.Authorization.Parameter));
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: #f4f4f4; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
 
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: white; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
            var col = SecurityTokenHandlerCollection.CreateDefaultSecurityTokenHandlerCollection();
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: #f4f4f4; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
            SecurityToken token = col.ReadToken(xmlReader);
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: white; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
 
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: #f4f4f4; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
            return token;
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: white; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
        }
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: #f4f4f4; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
 
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: white; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
        return null;
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: #f4f4f4; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
    }
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: white; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
}
~~~~

This implementation receives a “SecurityTokenHandlerCollection” instance
as part of the constructor. This class is part of WIF, and basically
represents a collection of token managers to know how to handle specific
xml authentication tokens (SAML is one of them).

I also created a set of extension methods for injecting these
interceptors as part of a service route when the service is initialized.

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: white; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
var basicAuthentication = new BasicAuthenticationInterceptor((u) => true, "ContactManager");
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: #f4f4f4; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
var samlAuthentication = new SamlAuthenticationInterceptor(serviceConfiguration.SecurityTokenHandlers);
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: white; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
 
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: #f4f4f4; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
// use MEF for providing instances
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: white; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
var catalog = new AssemblyCatalog(typeof(Global).Assembly);
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: #f4f4f4; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
var container = new CompositionContainer(catalog);
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: white; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
var configuration = new ContactManagerConfiguration(container);
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: #f4f4f4; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
 
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: white; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
RouteTable.Routes.AddServiceRoute<ContactResource>("contact", configuration, basicAuthentication, samlAuthentication);
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: #f4f4f4; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
RouteTable.Routes.AddServiceRoute<ContactsResource>("contacts", configuration, basicAuthentication, samlAuthentication);
~~~~

In the code above, I am injecting the basic authentication and saml
authentication interceptors in the “contact” and “contacts” resource
implementations that come as samples in the code preview.

I will use another post to discuss more in detail how the brokered
authentication with SAML model works with this new WCF Http bits.

The code is available to download in this
[location](/images/legacy/WCFHttpAuthentication.zip).

