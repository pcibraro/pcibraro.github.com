---
layout: post
title: "ActAs and OnBehalfOf support in WIF"
date: 2010-04-06
comments: true
categories: .NET
---

I
[discussed](http://weblogs.asp.net/cibrax/archive/2010/01/04/actas-in-ws-trust-1-4.aspx)
a time ago how WIF supported a new WS-Trust 1.4 element, “ActAs”, and
how that element could be used for authentication delegation.  The thing
is that there is another feature in WS-Trust 1.4 that also becomes handy
for this kind of scenario, and I did not mention in that last post,
“OnBehalfOf”.

Shiung Yong wrote an excellent summary about the difference of these two
new features in this forum
[thread](http://social.msdn.microsoft.com/Forums/en-US/Geneva/thread/78e3e93f-0e59-4236-a174-484212b66554).
He basically commented the following,

*“An ActAs RST element indicates that the requestor wants a token that
contains claims about two distinct entities: the requestor, and an
external entity represented by the token in the ActAs element.*

*An OnBehalfOf RST element indicates that the requestor wants a token
that contains claims only about one entity: the external entity
represented by the token in the OnBehalfOf element.*

*In short, ActAs feature is typically used in scenarios that require
composite delegation, where the final recipient of the issued token can
inspect the entire delegation chain and see not just the client, but all
intermediaries to perform access control, auditing and other related
activities based on the whole identity delegation chain. The ActAs
feature is commonly used in multi-tiered systems to authenticate and
pass information about identities between the tiers without having to
pass this information at the application/business logic layer.*

*OnBehalfOf feature is used in scenarios where only the identity of the
original client is important and is effectively the same as identity
impersonation feature available in the Windows OS today. When the
OnBehalfOf is used the final recipient of the issued token can only see
claims about the original client, and the information about
intermediaries is not preserved. One common pattern where OnBehalfOf
feature is used is the proxy pattern where the client cannot access the
STS directly but is instead communicating through a proxy gateway. The
proxy gateway authenticates the caller and puts information about him
into the OnBehalfOf element of the RST message that it then sends to the
real STS for processing. The resulting token is going to contain only
claims related to the client of the proxy, making the proxy completely
transparent and not visible to the receiver of the issued token.”*

Going back to WIF, “ActAs” and “OnBehalfOf” are both supported as
extensions methods in the WCF client channel.

public static class ChannelFactoryOperations \
{ \
  public static T CreateChannelActingAs\<T\>(this ChannelFactory\<T\>
factory, \
    SecurityToken actAs); \
  \
  public static T CreateChannelOnBehalfOf\<T\>(this ChannelFactory\<T\>
factory, \
    SecurityToken onBehalfOf); \
}

Both methods receive the security token with the identity of the
original caller.

