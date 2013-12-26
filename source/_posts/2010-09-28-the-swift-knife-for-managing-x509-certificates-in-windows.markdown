---
layout: post
title: "The swiss knife for managing X509 certificates in Windows"
date: 2010-09-28
comments: true
categories: 
---

[Raffaele Rialdi](http://www.iamraf.net), a security MVP from Italy, has
just released a very cool
[tool](http://www.iamraf.net/Tools/DeployManager-first-release-certificates-management)
to manage X509 certificates in windows. X509 certificates has always
represent a pain for most developers, as they are hard to deploy or
configure correctly with the right permissions. A tool like this is
absolutely need when working with frameworks like WCF or WIF that makes
an extensive use of certificates.

These are some of the features that you can find in this initial
version,

1.  Ability to create self-signed certificates from the UI by specifying
    all the different settings that you need (certificate name,
    expiration dates, password, certificate store).
2.  Ability to browse the different certificate stores and do things
    like checking the certificate details, the chain trust for an
    specific certificate, or change its ACL permissions.

You will able to find more details about the tool
[here](http://www.iamraf.net/Tools/DeployManager-first-release-certificates-management).

