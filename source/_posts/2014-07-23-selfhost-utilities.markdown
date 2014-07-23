---
layout: post
title: "SelfHost Utilities"
date: 2014-07-23 09:49
comments: true
categories: webapi
---

Self Hosting a Http server is a very common scenario these days with the push that Microsoft and the rest of the community are giving to Owin. One of the challenges you often find in this scenario is the ability to use HTTPS, and I can say by experience that it's not something trivial. You have to run several commands, and usually generate a self signed certificate for SSL. 

As part of the project where I was working on, we had to automate many of these steps in the installation process so we came up with a set of utilities classes that call the underline Win32 APIS for generate the certificate and also do the required registrations for the namespace and port. The process for doing this with these classes is pretty straigforward as it is shown below,

```csharp
var cert = X509Util.CreateSelfSignedCertificate(Environment.MachineName);

//Register a namespace reservation for everyone in localhost in port 9010
HttpServerApi.ModifyNamespaceReservation(new Uri("https://localhost:9010"), 
  "everyone", 
  HttpServerApiConfigurationAction.AddOrUpdate);
        
//Register the SSL certificate for any address (0.0.0.0) in the port 9010.
HttpServerApi.ModifySslCertificateToAddressBinding("0.0.0.0", 
  9010, 
  cert.GetCertHash(), 
  System.Security.Cryptography.X509Certificates.StoreName.My, 
  HttpServerApiConfigurationAction.AddOrUpdate);
```

All the code is now available for you in github [SelfHostUtilities](https://github.com/pcibraro/SelfHostUtilities).