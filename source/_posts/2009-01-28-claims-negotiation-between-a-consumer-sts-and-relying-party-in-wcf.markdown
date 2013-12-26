---
layout: post
title: "Claims negotiation between a consumer, STS and Relying Party in WCF"
date: 2009-01-28
comments: true
categories: Federation
---

According to the WS-Trust specification, a service consumer has a way to
negotiate or ask for specific claims to the STS. Those claims (or some
of them) will be generally used by  the service implementation running
on the relying party.

They are negotiated through an "claims" element in the RST message,

\<wst:RequestSecurityToken xmlns:wst="..."\>

        \<wst:TokenType\>...\</wst:TokenType\>

        \<wst:RequestType\>...\</wst:RequestType\>

        ...

        \<wsp:AppliesTo\>...\</wsp:AppliesTo\>

        **\<wst:Claims Dialect="..."\>...\</wst:Claims\>**

        \<wst:Entropy\>

              \<wst:BinarySecret\>...\</wst:BinarySecret\>

         \</wst:Entropy\>

        \<wst:Lifetime\>

            \<wsu:Created\>...\</wsu:Created\>

            \<wsu:Expires\>...\</wsu:Expires\>

        \</wst:Lifetime\>

\</wst:RequestSecurityToken\>

*The "wst:claims" is an optional element for requesting a specific set
of claims. Typically, this element contains required and/or optional
claim information identified in a service's policy.*

Based on these facts, we can elaborate some possible scenarios for
claims negotiation between these three parties.

​1. **No negotiation at all**

The STS might just ignore these claims requirements in the RST message
and always returns a fixed claim set according to the consumer identity,
or the service might not express what claims it expects at all. This
scenario might be suitable for a local STS in small-sized or
medium-sized organizations, where the IT department has a complete
control over the client applications and services that interact with
that STS. This kind of solution is easier to implement, and quite rigid
too, a change in the claims required by the service will also require
changes in the STS implementation. As you see, this solution does not
scale at all for a high number of applications or relying party
services.

Many of the STS examples you will find today are implemented like this.

​2. **Negotiation based on the AppliesTo header**.

This solution present a subtle difference with the one discussed before,
the claims vary according the relying party that will make use of them.
The STS ignores the claims requirements in the RST messages and returns
a claim set based on the received AppliesTo header. An existing
agreement must exist between the STS and the relying party, which will
include in addition to the key for encrypting the tokens, a number of
expected claims.  Again, easy to implement, difficult to scale up.

​3. **Manual negotiation based on the "Claims" header**.

In this scenario, the consumer sends the expected claims in the "claims"
header and the STS makes use of them for generating the resulting token.
However, the negotiation of those claims between the consumer and the
relying party is manual, a previous agreement must exist, the service
does not express those requirements through metadata. This means that
the claims are hard-coded during development in the client
configuration.  If the service requires additional claims, only the
client configuration will have to be changed, the STS does not have to
be touched at all.

If you are implementing a custom STS with the latest Microsoft Geneva
bits, there is a property "Claims" in the RequestSecurityToken for
getting access to these values.

protected override IClaimsIdentity
GetOutputClaimsIdentity(IClaimsPrincipal principal, RequestSecurityToken
request, Scope scope)

{

    IClaimsIdentity outputIdentity = new ClaimsIdentity();

 

    foreach (Claim claim in request.Claims)

    {

        //Do something...

 

        outputIdentity.Claims.Add(...);

    }

 

    return outputIdentity;

}

The client can specify those claims through configuration as well,

\<wsFederationHttpBinding\>

  \<binding name="ServiceBinding"\>

    \<security mode="Message"\>

      \<message
issuedTokenType="http://docs.oasis-open.org/wss/oasis-wss-saml-token-profile-1.1\#SAMLV1.1"
negotiateServiceCredential="false"\>

        \<claimTypeRequirements\>

          \<add claimType
="http://schemas.microsoft.com/ws/2005/05/identity/claims/EmailAddress"/\>

          \<add claimType
="http://schemas.microsoft.com/ws/2005/05/identity/claims/GivenName"/\>

          \<add claimType
="http://schemas.microsoft.com/ws/2005/05/identity/claims/Surname"
isOptional ="true"/\>

        \</claimTypeRequirements\>

        \<issuer\>\</issuer\>

      \</message\>

    \</security\>

  \</binding\>

\</wsFederationHttpBinding\>

Once they are added to the binding configuration, WCF will automatically
include them as part of the RST message to the STS.

​4. **Automatic negotiation based on the "Claims" header**.

This is by far the best solution we can find. The three parties
automatically negotiates the claims at runtime,

I. The service exposes the claim requirements through metadata
(WS-Policy)

​II. The client acquires the service's policy and requirements using
some mechanism that could be WS-MetatadaExchange.  Later,  the client
includes some claim requirements into the RST message that will be send
to the STS.

​III. The STS extracts those requirements from the RST message, and
then, it makes use of them for generating the resulting token.

The Cardspace identity selector on the consumer side works like this. It
first detects what claims are needed by the Relying Party, and then,
displays all the possible cards (From different Identity providers) that
satisfy those requirements to the user.

Exposing the claim requirements on the relying party through WCF is
equivalent to do it on the client side (same binding configuration),

\<wsFederationHttpBinding\>

  \<binding name="ServiceBinding"\>

    \<security mode="Message"\>

      \<message
issuedTokenType="http://docs.oasis-open.org/wss/oasis-wss-saml-token-profile-1.1\#SAMLV1.1"\>

        \<claimTypeRequirements\>

          \<add claimType
="http://schemas.microsoft.com/ws/2005/05/identity/claims/EmailAddress"/\>

          \<add claimType
="http://schemas.microsoft.com/ws/2005/05/identity/claims/GivenName"/\>

          \<add claimType
="http://schemas.microsoft.com/ws/2005/05/identity/claims/Surname"
isOptional ="true"/\>

        \</claimTypeRequirements\>

        \<issuer\>\</issuer\>

      \</message\>

    \</security\>

  \</binding\>

\</wsFederationHttpBinding\>

