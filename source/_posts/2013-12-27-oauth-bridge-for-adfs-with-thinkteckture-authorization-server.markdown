---
layout: post
title: "OAuth Bridge for ADFS with ThinkTecture Authorization Server"
date: 2013-12-27 10:23
comments: true
categories: 
---

ADFS 2.0 only supports SAML 2.0 and WS-Federation for supporting single sign on web applications and services. However, some new development platforms such as iOS only support OAuth 2.0, which makes the use of ADFS a bit tricky. ADFS for Windows Server 2012 R2 already provides some limited support for JSON Web Tokens (JWT) with the OAuth 2.0 the code flow (As it is described in this [excellent post](http://www.cloudidentity.com/blog/2013/07/30/securing-a-web-api-with-windows-server-2012-r2-adfs-and-katana/) by Vittorio). 

For some other scenarios in which your applications or services rely on JWTs for doing authentication/authorization, the Thinktecture Authorization Server represents a very nice alternative, which integrates really well with ADFS. The Authorization Server can act as a brige or broker between client apps that use OAuth 2.0 to get a JWT, and ADFS, which is used for authenticating the users.

The configuration of the Authorization Server is very simple. Once you get Authorization Server deployed in IIS, it uses Entity Framework code first to automatically generate the backend database for you on the first use. After that, you only need to change the configuration files to trust ADFS as it is shown bellow,

[Authorization Server Folder]\Configuration\identityModel.config

```xml
<issuerNameRegistry type="System.IdentityModel.Tokens.ValidatingIssuerNameRegistry, 
	System.IdentityModel.Tokens.ValidatingIssuerNameRegistry">
	<authority name="http://[your ADFS server]/adfs/services/trust">
    	<keys>
			<add thumbprint="[signing cert thumprint]"/>
        </keys>
        <validIssuers>
          <add name="http://[your ADFS server]/adfs/services/trust" />
        </validIssuers>
	</authority>
</issuerNameRegistry>
```

[Authorization Server Folder]\Configuration\identityModel.services.config

```xml
<system.identityModel.services>
	<federationConfiguration>
		<wsFederation passiveRedirectEnabled="true"
 issuer="https://[your ADFS server]/adfs/ls/" realm="urn:authorizationserver" />
 	</federationConfiguration>
</system.identityModel.services>
```

urn:authorizationserver is the Relying Party identifier I used for configuring Authorization Server in ADFS. In ADFS, you only to configure a new relying party with a WS-Federation endpoint, and set the URL of the Authorization Server. Make also sure the Relying Party identifier matches the one you configured in Authorization Server.

The following code shows how to get a JWT from Authorization Server using the Resource Owner Flow (The user is authenticated by Authorization Server in ADFS)

```csharp
private static string GetToken()
{
    ServicePointManager.ServerCertificateValidationCallback = 
    	(o, cert, chain, ssl) => true;

    HttpClient client = new HttpClient();

    HttpRequestMessage request = new HttpRequestMessage(HttpMethod.Post, 
    	"https://[your auth server]/[App namespace in Authorization Server]/oauth/token");
    request.Headers.Add("Authorization", 
    	"Basic " + Convert.ToBase64String(System.Text.ASCIIEncoding.ASCII.GetBytes(
    		string.Format("{0}:{1}", "[client id]", "[client secret]"))));
    
    request.Content = new FormUrlEncodedContent(new Dictionary<string, string>
    {
         {"grant_type", "password"},
         {"username", "[account]"}, //Windows account name without domain
         {"password", "[password]"},
         {"scope", "All"}
    });

    HttpResponseMessage response = client.SendAsync(request).Result;
    response.EnsureSuccessStatusCode();
    JObject json = JObject.Parse(response.Content.ReadAsStringAsync().Result);

    string accessToken = json["access_token"].ToString();
    string refreshToken = json["refresh_token"].ToString();

    return accessToken;
}
```

You can find more details in the [Dominick's blog](http://leastprivilege.com/2013/09/19/adding-oauth2-to-adfs-and-thus-bridging-the-gap-between-modern-applications-and-enterprise-back-ends)

