---
layout: post
title: "Creating X509 Certificates for WSE or WCF"
date: 2006-08-08
comments: true
categories: WCF
---

What certificate to use or where to get one of these, is perhaps is one
the most common question I read often in the MS newsgroups.\
Well, for starters, there are many kinds of X509 certificates around,
each one for a different purpose. And to make the matters worse, each
vendor offers similar certificates with different names.\
[Verisign](http://www.verisign.com/ "Verisign"),
[RSA](http://www.rsasecurity.com/ "RSA"),
[Thawte](http://www.thawte.com/ "Thawte") and others usually offer
certificates for the following purposes:

+--------------------+--------------------+--------------------+--------------------+
| **Purpose**        | **Description**    | **Data Signing     |  **Key Exchange    |
|                    |                    | Support**          | support**          |
+--------------------+--------------------+--------------------+--------------------+
| Code Signing       | A code signing     | Yes                | No                 |
|                    | certificate is     |                    |                    |
|                    | usually            |                    |                    |
|                    | recommended for    |                    |                    |
|                    | any publisher who  |                    |                    |
|                    | plans to           |                    |                    |
|                    | distribute code or |                    |                    |
|                    | content over the   |                    |                    |
|                    | Internet or        |                    |                    |
|                    | corporate          |                    |                    |
|                    | extranets and      |                    |                    |
|                    | needs to assure    |                    |                    |
|                    | the integrity and  |                    |                    |
|                    | authorship of that |                    |                    |
|                    | code               |                    |                    |
+--------------------+--------------------+--------------------+--------------------+
| E-Mail Security    | An e-mail          | Yes                | No                 |
|                    | certificate can be |                    |                    |
|                    | used to digitally  |                    |                    |
|                    | sign email         |                    |                    |
|                    | messages or        |                    |                    |
|                    | to encrypt email   |                    |                    |
|                    | contents and       |                    |                    |
|                    | attachments.       |                    |                    |
+--------------------+--------------------+--------------------+--------------------+
| SSL                | A SSL certificate  | Yes                | No                 |
|                    | is usually         |                    |                    |
|                    | configured in a    |                    |                    |
|                    | web site to offer  |                    |                    |
|                    | SSL protection     |                    |                    |
|                    | (HTTPS)            |                    |                    |
+--------------------+--------------------+--------------------+--------------------+

 

 

The WSE's turn-key assertions always require certificates with data
signing and key exchange support. The data signing support is used to
generate XML signatures and the key exchange function is used to 
encrypt data (It actually creates a EncryptedKey, for more information
about this, read this [blog
entry](/cibrax/archive/2005/10/04/426585.aspx "blog entry")). However, a
custom security assertion can be created to use different certificates
for data signing and data encryption. I did something like that in one
of the projects where I worked, you can take a look to the code in this
[GDN
workspace](http://www.gotdotnet.com/codegallery/codegallery.aspx?id=0fecd2c7-b2b1-4d85-bd66-9d07a6ecbd86 "GDN workspace")\
On the other hand, WCF allows to specify different certificates for data
signing and key interchange by means of the X509 Security Token
providers. \
Another approach and probably most attractive in many organizations is
to create custom X509 certificates using an in-house certificate
authority. Microsoft comes with two solutions out of the box to generate
certificates, the tool "MakeCert.exe" to create test certificates for
development environments and a more robust solution, Microsoft
Certificate Server (MCS), to configure a real PKI in the organization.

**Creating Certificates with MCS**

In a nutshell, Microsoft Certificate Server is a product that allows to
mount a real PKI in any organization. This product can run in two modes,
Active directory integrated or stand alone. This last one is
probably useful for software development organizations or independent
consultants working on products that require certificates.\
Microsoft also provides a component **CertEnroll** to automate the
process of creating and asking certificates to the
certificate authority.\
In the following paragraphs I will show how to use this component
to create a valid X509 certificate to be used in WSE or WCF. 

The creation of a certificate in MCS involves two steps:

​1. Send a certificate request to the Certificate Authority\
2. Obtain the certificate from the Certificate Authority after
the request was authorized

The CertEnroll COM contains methods to perform both steps, still some
lines of code are required. The code below illustrates how to request
and obtain a certificate from a CA (I wrote the code in vbscript).

  --------------------------------------------------------------------------------------------
  \
  Option Explicit \
  const CERT\_SYSTEM\_STORE\_LOCAL\_MACHINE = &H20000\
  const CRYPT\_EXPORTABLE = 1 \
  \
  Const AT\_KEYEXCHANGE = 1 \
  Const AT\_SIGNATURE = 2 \
  Const CRYPT\_ARCHIVABLE = &H000004000 \
  Const RSA1024BIT\_KEY = &H04000000 \
  \
  Const XECR\_PKCS10\_V2\_0 = 1 \
  Const XECR\_PKCS7 = 2 \
  Const XECR\_CMC = 3 \
  Const XECR\_PKCS10\_V1\_5 = 4 \
  \
  Const CR\_OUT\_BASE64HEADER = 0 \
  Const CR\_OUT\_BASE64 = 1 \
  Const CR\_OUT\_BINARY = 2 \
  Const CR\_OUT\_CHAIN = &h100 \
  Const CR\_IN\_BASE64HEADER = 0 \
  Const CR\_IN\_BASE64 = &H1 \
  Const CR\_IN\_PKCS10 = &H100 \
  Const CR\_IN\_KEYGEN = &H200 \
  Const CR\_IN\_ENCODEANY = &Hff \
  Const CR\_IN\_FORMATANY = 0 \
  \
  Const CR\_DISP\_INCOMPLETE = 0 \
  Const CR\_DISP\_ERROR = 1 \
  Const CR\_DISP\_DENIED = 2 \
  Const CR\_DISP\_ISSUED = 3 \
  Const CR\_DISP\_ISSUED\_OUT\_OF\_BAND = 4 \
  Const CR\_DISP\_UNDER\_SUBMISSION = 5 \
  Const CR\_DISP\_REVOKED = 6 \
  \
  \
  Const PROPTYPE\_LONG = 1 \
  Const PROPTYPE\_DATE = 2 \
  Const PROPTYPE\_BINARY = 3 \
  Const PROPTYPE\_STRING = 4 \
  \
  \
  Const CR\_GEMT\_HRESULT\_STRING = 1 \
  \
  \
  Const FR\_PROP\_NONE = 0 ' Invalid \
  Const FR\_PROP\_FULLRESPONSE = 1 ' Binary \
  Const FR\_PROP\_STATUSINFOCOUNT = 2 ' Long \
  Const FR\_PROP\_BODYPARTSTRING = 3 ' String, Indexed \
  Const FR\_PROP\_STATUS = 4 ' Long, Indexed \
  Const FR\_PROP\_STATUSSTRING = 5 ' String, Indexed \
  Const FR\_PROP\_OTHERINFOCHOICE = 6 ' Long, Indexed \
  Const FR\_PROP\_FAILINFO = 7 ' Long, Indexed \
  Const FR\_PROP\_PENDINFOTOKEN = 8 ' Binary, Indexed \
  Const FR\_PROP\_PENDINFOTIME = 9 ' Date, Indexed \
  Const FR\_PROP\_ISSUEDCERTIFICATEHASH =10 ' Binary, Indexed \
  Const FR\_PROP\_ISSUEDCERTIFICATE =11 ' Binary, Indexed \
  Const FR\_PROP\_ISSUEDCERTIFICATECHAIN =12 ' Binary, Indexed \
  Const FR\_PROP\_ISSUEDCERTIFICATECRLCHAIN=13 ' Binary, Indexed \
  Const FR\_PROP\_ENCRYPTEDKEYHASH =14 ' Binary, Indexed \
  Const FR\_PROP\_FULLRESPONSENOPKCS7 =15 ' Binary \
  \
  \
  Const KeyLength = 1024 \
  \
  \
  Dim CertEnroll \
  \
  \
  Set CertEnroll = CreateObject( "CEnroll.CEnroll" ) \
  \
  \
  Dim RequestStr, CertRequest, Disposition, ID \
  \
  \
  CertEnroll.ProviderName = "Microsoft Enhanced Cryptographic Provider v1.0" \
  CertEnroll.KeySpec = AT\_KEYEXCHANGE \
  CertEnroll.GenKeyFlags = CRYPT\_EXPORTABLE \
  \
  \
  RequestStr = CertEnroll.createRequest( XECR\_CMC, "CN=John Smith", "1.3.6.1.5.5.7.3.2" ) \
  \
  \
  Set CertRequest = CreateObject( "CertificateAuthority.Request" ) \
  Disposition = CertRequest.Submit( \_ \
  CR\_IN\_ENCODEANY Or CR\_IN\_FORMATANY, \_ \
  RequestStr, \_ \
  "", \_ \
  "Localhost\\MyCertAuth" ) \
  \
  \
  ID = CertRequest.GetRequestId \
  \
  \
  MsgBox ID \
  \
  --------------------------------------------------------------------------------------------

 

In a few words, the code above submits a certificate request to a MCS
service running on "Localhost\\MyCertAuth". The certificate distinguish
name will be "CN=Jonh Smith" and the long number "1.3.6.1.5.5.7.3.2"
only describes the purpose of the certificate. (MCS provides a set of
templates, each one with a different number identifier). \
It is important to store the RequestId in some place since it will be
necessary later to retrieve the certificate from the CA (Once the
request was authorized by one of the CA administrators).

 

  -----------------------------------------------------------------------------------
  Option Explicit \
  \
  const CERT\_SYSTEM\_STORE\_LOCAL\_MACHINE = &H20000 \
  const CRYPT\_EXPORTABLE = 1 \
  \
  Const AT\_KEYEXCHANGE = 1 \
  Const AT\_SIGNATURE = 2 \
  Const CRYPT\_ARCHIVABLE = &H000004000 \
  Const RSA1024BIT\_KEY = &H04000000 \
  \
  Const XECR\_PKCS10\_V2\_0 = 1 \
  Const XECR\_PKCS7 = 2 \
  Const XECR\_CMC = 3 \
  Const XECR\_PKCS10\_V1\_5 = 4 \
  \
  \
  Const CR\_OUT\_BASE64HEADER = 0 \
  Const CR\_OUT\_BASE64 = 1 \
  Const CR\_OUT\_BINARY = 2 \
  Const CR\_OUT\_CHAIN = &h100 \
  \
  Const CR\_IN\_BASE64HEADER = 0 \
  Const CR\_IN\_BASE64 = &H1 \
  Const CR\_IN\_PKCS10 = &H100 \
  Const CR\_IN\_KEYGEN = &H200 \
  Const CR\_IN\_ENCODEANY = &Hff \
  Const CR\_IN\_FORMATANY = 0 \
  \
  \
  Const CR\_DISP\_INCOMPLETE = 0 \
  Const CR\_DISP\_ERROR = 1 \
  Const CR\_DISP\_DENIED = 2 \
  Const CR\_DISP\_ISSUED = 3 \
  Const CR\_DISP\_ISSUED\_OUT\_OF\_BAND = 4 \
  Const CR\_DISP\_UNDER\_SUBMISSION = 5 \
  Const CR\_DISP\_REVOKED = 6 \
  \
  \
  Const PROPTYPE\_LONG = 1 \
  Const PROPTYPE\_DATE = 2 \
  Const PROPTYPE\_BINARY = 3 \
  Const PROPTYPE\_STRING = 4 \
  \
  \
  Const CR\_GEMT\_HRESULT\_STRING = 1 \
  Const FR\_PROP\_NONE = 0 ' Invalid \
  Const FR\_PROP\_FULLRESPONSE = 1 ' Binary \
  Const FR\_PROP\_STATUSINFOCOUNT = 2 ' Long \
  Const FR\_PROP\_BODYPARTSTRING = 3 ' String, Indexed \
  Const FR\_PROP\_STATUS = 4 ' Long, Indexed \
  Const FR\_PROP\_STATUSSTRING = 5 ' String, Indexed \
  Const FR\_PROP\_OTHERINFOCHOICE = 6 ' Long, Indexed \
  Const FR\_PROP\_FAILINFO = 7 ' Long, Indexed \
  Const FR\_PROP\_PENDINFOTOKEN = 8 ' Binary, Indexed \
  Const FR\_PROP\_PENDINFOTIME = 9 ' Date, Indexed \
  Const FR\_PROP\_ISSUEDCERTIFICATEHASH =10 ' Binary, Indexed \
  Const FR\_PROP\_ISSUEDCERTIFICATE =11 ' Binary, Indexed \
  Const FR\_PROP\_ISSUEDCERTIFICATECHAIN =12 ' Binary, Indexed \
  Const FR\_PROP\_ISSUEDCERTIFICATECRLCHAIN=13 ' Binary, Indexed \
  Const FR\_PROP\_ENCRYPTEDKEYHASH =14 ' Binary, Indexed \
  Const FR\_PROP\_FULLRESPONSENOPKCS7 =15 ' Binary \
  \
  \
  Const KeyLength = 1024 \
  \
  \
  Dim CertEnroll, CertRequest \
  Dim Disposition \
  \
  \
  Set CertEnroll = CreateObject( "CEnroll.CEnroll" ) \
  Set CertRequest = CreateObject( "CertificateAuthority.Request" ) \
  Disposition = CertRequest.GetIssuedCertificate("localhost\\MyCertAuth", 24, "") \
  \
  \
  If Disposition = CR\_DISP\_ISSUED Then \
  \
  Dim Cert \
  \
  Cert = CertRequest.GetFullResponseProperty( \_ \
  \
  FR\_PROP\_FULLRESPONSE, \_ \
  \
  0, \_ \
  \
  PROPTYPE\_BINARY, \_ \
  \
  CR\_OUT\_BASE64 ) \
  \
  \
  \
  Dim sCert \
  \
  sCert = CertEnroll.getCertFromResponse(Cert) \
  \
  \
  \
  CertEnroll.createFilePFX "password", "c:\\cert.pfx" \
  Else \
  \
  WScript.echo "Disposition = " + cstr( Disposition ) \
  \
  If CertRequest.GetLastStatus \<\> 0 Then \
  \
  \
  \
  WScript.echo "Error : " + cstr( CertRequest.GetErrorMessageText( \_ \
  \
  \
  \
  CertRequest.GetLastStatus, CR\_GEMT\_HRESULT\_STRING )) \
  \
  \
  End If \
  End If \
  -----------------------------------------------------------------------------------

 

The code above retrieves the certificate for the request id 24 (It is
only an example, the real id should be used there) and stores it on the
file "c:\\cert.pfx" (By the way, this is not a good practice since the
certificate can be copied).

 

**Creating Certificates with MakeCert**

 

MakeCert is a tool provided by Microsoft to create test certificates
that can be used during the development of a product (For developing and
testing purposes only). These certificates have also performance
problems, certain cryptographic operations may perform slowly when they
are used. Certificates issued from a true Certificate Authority do not
have this problem, and it is a know issue.

In order to create a certificate with this tool, the following arguments
must be considered:

- sr: Store Location, it can be LocalMachine or CurrentUser\
- ss: Store Folder, it can take different values but these are probably
the most common, My (Personal) or Trusted (Trusted Folder).\
- n: Certificate Distinguished name. It is very import to chose a right
name for the certificate since it will identify it. (This name is also
used to look for the certificate)

For example,

makecert.exe -sr LocalMachine -ss My -a sha1 -n CN=MyServerCert -sky
exchange -pe 

This tool only generates and stores the certificate in the certificate
store, but it does not assign any permission on that certificate. If you
planning to use the certificate from WSE or WSE, you may need to give
read permission to the account running the service, for instance the
account ASPNET for a normal web service.\
Microsoft provides another tool to grant permissions on certificates,
the name of this tool is **winhttpcertcfg**.\
The following sample, grant permission on the certificate created above
to the ASPNET account:

winhttpcertcfg -g -c LOCAL\_MACHINE\\My -s MyServerCert -a ASPNET

