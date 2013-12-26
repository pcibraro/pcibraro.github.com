---
layout: post
title: "“The system cannot find the file specified” error in the WIF FAM module"
date: 2010-02-17
comments: true
categories: .NET
---

The Federation Authentication Module (FAM) shipped as part of WIF
protects by the default the session cookies from being tampered with in
passive scenarios using DPAPI. As I mentioned in the past, this
technique simplifies a lot the initial deployment for the whole solution
as nothing extra needs to configured, the automatically generated DPAPI
key is used to protect the cookies, so this might be reason to have that
as default protection mechanism in WSE, WCF and now WIF.

However, this technique has some serious drawbacks from my point of view
that makes it useless for real enterprise scenarios.

If web application that relies on FAM for authenticating the users is
hosted in IIS. The account running the IIS process needs to have a
profile created in order to use DPAPI. A workaround for this is to log
into the machine with that account to create the initial profile or run
some script to do it automatically.

DPAPI is not suitable for web farm scenarios, as the machine key is used
to protect the cookies. If the cookie is protected with one key, the
following requests must be sent to the same machine. A workaround for
this could be to use sticky sessions, so all the user requests from the
same machine are handled by the same machine on the farm.

Fortunately, WIF already provides some built-in classes to replace this
default mechanism by a protection mechanism based on RSA keys with X509
certificates.

The “SecuritySessionHandler” is the handler in WIF that is responsible
for tracking the authentication sessions into a cookie. That handler
receives by default some built-in classes that applies transformations
to the cookie content, such as the DeflatCookieTransform and the
ProtectedDataCookieTransform (for protecting the content with DPAPI).
There are also two other CookieTransform derived classes that are not
used at all, and becomes very handy to enable enterprise scenarios, the
RSAEncryptionCookieTransform and RSASignatureCookieTransform classes.
Both classes receives either a RSA key or X509 certificate that is used
to encrypt or sign the cookie content.

Therefore, you can put the following code in the global.asax file to
replace the default cookie transformations by the ones that use a X509
certificate.

protected void Application\_Start(object sender, EventArgs e) \
        { \
            FederatedAuthentication.ServiceConfigurationCreated += new
EventHandler\<Microsoft.IdentityModel.Web.Configuration.ServiceConfigurationCreatedEventArgs\>(FederatedAuthentication\_ServiceConfigurationCreated);
\
            \
        }

        void FederatedAuthentication\_ServiceConfigurationCreated(object
sender,
Microsoft.IdentityModel.Web.Configuration.ServiceConfigurationCreatedEventArgs
e) \
        { \
            var cookieProtectionCertificate =
CertificateUtil.GetCertificate(StoreName.My, \
                StoreLocation.LocalMachine, "CN=myTestCert"); \
             \
            e.ServiceConfiguration.SecurityTokenHandlers.AddOrReplace( \
                new SessionSecurityTokenHandler(new
System.Collections.ObjectModel.ReadOnlyCollection\<CookieTransform\> ( \
                    new List\<CookieTransform\> \
                    { \
                        new DeflateCookieTransform(), \
                        new
RsaEncryptionCookieTransform(cookieProtectionCertificate), \
                        new
RsaSignatureCookieTransform(cookieProtectionCertificate) \
                    }) \
                )); \
        }

