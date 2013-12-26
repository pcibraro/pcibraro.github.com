---
layout: post
title: "Carrying sensitive information in SAML assertions"
date: 2009-02-24
comments: true
categories: .NET
---

When SAML is used in conjunction with WS-Security, only an small piece
of the token is encrypted, the proof key for the relying party. The rest
of the token goes in plain text, that also includes the user's claims.

\<saml:Assertion\> 

  \<saml:Conditions NotBefore="2009-02-24T19:48:20.500Z"
NotOnOrAfter="2009-02-24T19:53:20.500Z"\>\</saml:Conditions\>

  \<saml:AttributeStatement\>

  \<saml:Subject\>

    \<saml:NameIdentifier\>\</saml:NameIdentifier\>

    \<saml:SubjectConfirmation\>

     
\<saml:ConfirmationMethod\>urn:oasis:names:tc:SAML:1.0:cm:holder-of-key\</saml:ConfirmationMethod\>

        \<KeyInfo
xmlns="http://www.w3.org/2000/09/xmldsig\#"\>...\</KeyInfo\>

   \</saml:SubjectConfirmation\>

  \</saml:Subject\>

  \<saml:Attribute AttributeName="displayName"
AttributeNamespace="http://schemas.microsoft.com/xsi/2005/05/role"\>

**      \<****saml:AttributeValue\>John Foo\</saml:AttributeValue****\>
\<--Attribute value--\>**

  \</saml:Attribute\>

  \</saml:AttributeStatement\>

  \<Signature
xmlns="http://www.w3.org/2000/09/xmldsig\#"\>...\</Signature\>

\</saml:Assertion\>

Knowing this, you should never include sensitive information as claims
in a SAML token. This is also related to the [identity law \#2, "Minimal
Disclosure for a Constrained
Use"](http://www.identityblog.com/stories/2004/12/09/thelaws.html). The
Identity provider should only disclose the least amount of identifiying
information for executing the operation on the relying party.

Some examples are,

-   *A winery only needs to know whether the customer is in a legal age
    for buying alcohol according to the**law, a claim like "over21"
    should be enough for that purpose, there is not need to know the
    customer birth**date at all.*
-   *An online store that sells products does not necessary need to know
    the number of every credit card owned by a customer, a friendly name
    representing the card and optionally the available balance could be
    enough for completing a purchase.*

SAML 2.0 introduces the concept of "encrypted attribute", which clearly
states its purpose, encrypt individual assertions in a SAML token. In
this way, a token can now carry the encrypted proof key and optionally
one or more encrypted assertions with sensitive information.

You can take a look at [this
page](https://spaces.internet2.edu/display/SHIB/SAMLDiffs) for more
information about the differences between SAML 1.1 and 2.0.

Geneva Framework Beta 1 already implements a subset of SAML 2.0,
however, it looks like this feature has been left out in the current
release. Not sure either whether this feature will be included as part
of the final release (Last quarter of 2009). I created [a
post](http://social.msdn.microsoft.com/Forums/en-US/Geneva/thread/a63e31cb-a109-4e99-8538-2eb084ed3827)
in the forums some time ago, I haven't received any feedback yet.

