---
layout: post
title: "Automatic Client Cert Detection in ADFS 2.0"
date: 2013-11-22 12:29
comments: true
categories: adfs
---

ADFS 2.0 supports multiple authentication methods through authentication handlers that are mutually exclusive. If one of the handlers runs, the others don't. There is no way to implement a fallback logic if one the handlers fail to run or the user was not able to provide the expected credentials, so supporting dual authentication such as username/password and client certificates is sometimes problematic.  The following list shows the handlers included out of the box with ADFS 2.0,

```xml
<microsoft.identityServer.web>
	<localAuthenticationTypes>
		<add name="Forms" page="FormsSignIn.aspx" />
		<add name="Integrated" page="auth/integrated/" />
		<add name="TlsClient" page="auth/sslclient/" />
		<add name="Basic" page="auth/basic/" />
	</localAuthenticationTypes>
<microsoft.identityServer.web>
```

Those handlers are for Forms Authentication (Username/Password in an html form), Integrated Windows Authentication, Client Cert Authentication, and finally Http Basic Auth Authentication. The order in this list determines the priority by default unless the priority has been changed in the execution context. The order in the execution context can be changed in multiple ways. For example, when you are doing WS-Federation, the relying party can pass an additional "WAuth" query string parameter with the expected authentication type such as "urn:oasis:names:tc:SAML:1.0:am:password" for Forms Authentication or "urn:ietf:rfc:2246" for Client Cert authentication, which overrides the priority in the list set on the configuration. The same thing can be done for SAML 2.0 by changing the authentication context in the SAML Request message or changing the URL. 

However, as I said before, in one of the handlers run, you will not have a chance to run any of the other handlers unless you implement some workaround. For example, you might want to add a checkbox in the html form for Forms authentication to allow the user to authenticate with a client certificate. When the checkbox is clicked, you do an http redirect to ADFS but changing the URL this time to use this new authentication method. If the RP is using WS-Federation, that means a new redirect that includes the WAuth query string variable with the value "urn:ietf:rfc:2246", or a redirect to "auth/sslclient/" in the case of SAML 2.0.

If you want to avoid this manual step and detect the client certificate automatically, more work is involved, and it is what we are going to discuss next.

1. Enable Forms Authentication as default in the configuration file. The "Forms" handler must be on top of the list.
2. Create a new virtual directory in the ADFS IIS, and configure that virtual directory to require HTTPS and Accept Client Certs.

![IIS Settings](/images/adfs_cert/iis.png "IIS Settings")

3. Configure an ASP.NET Generic handler in that virtual directory to check if the certificate is present or not

```c#
using System;
using System.Web;

public class CertificateHandler : IHttpHandler
{

	public void ProcessRequest(HttpContext context)
    {
        context.Response.ContentType = "application/javascript";

        if (context.Request.ClientCertificate.IsPresent)
        {
            context.Response.Write("if(typeof(certificateCallback) == 'function') 
            	{ (certificateCallback) (true); }");
        }
        else
        {
            context.Response.Write("if(typeof(certificateCallback) == 'function') 
            	{ (certificateCallback) (false); }");
        }
        
    }

    public bool IsReusable
    {
        get
        {
            return false;
        }
    }

}
```

This code checks if the client certificate is present and invokes a javascript callback with "true" or "false" based on that condition. This code does not check the authenticity of the cert or anything else, and it is only for determining if the certificate is present. The rest of the validations will be done by ADFS.

4. Configure this handler in the web.config of that virtual directory.

```xml
<system.webServer>
	<handlers>
	  <add name="certificate" type="CertificateHandler" path="Script" verb="GET"/>
	</handlers>
</system.webServer>
```

5. Include a CertCallBack.js javascript file into the ADFS folder with the code required for the callback,

```javascript
window.certificateCallback = function (cert) {
    var url;
    if (cert) {
        url = "auth/sslclient/" + window.location.search;
        document.location.href = url;
    }
};
```

This code is very simple. It simply does a redirect when the cert is present (the callback was called with a "true" value). The redirect in this sample is only valid for SAML 2.0

6. Modify the FormsSign.aspx page to include the following code at the end of the Page_Load event,

```c#
string script = System.IO.File.ReadAllText(Server.MapPath("CertCallBack.js"));

this.Page.ClientScript.RegisterClientScriptBlock(this.GetType(), "callback", script, true);
this.Page.ClientScript.RegisterClientScriptInclude("certauth", "/CertDetection/Script");
```

This code injects our CertCallBack.js script in the page and it also includes the script located in the virtual directory configured to accept client certs (/CertDetection/Script invokes the ASP.NET Handler basically)

That should be all. If the browser detects a client certificate when the script in the CertDetection virtual directory is resolved, the callback will be executed with a true value, making the form to do a redirect to authenticate the client with a Client Certificate instead. Otherwise, the client will see the Form to enter the username and password. We are adding a new virtual directory for not interferring with the existing authentication methods in the ADFS virtual directory.

The code for the handler and the javascript callback is included in the [attached zip file](/external/adfs_cert/CertDetection.zip)


 

