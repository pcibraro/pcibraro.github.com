---
layout: post
title: "Negotiating SAML tokens for REST clients with the HttpClient class"
date: 2009-03-18
comments: true
categories: Geneva
---

Continuing my post [“Brokered authentication for REST active
clients”](http://weblogs.asp.net/cibrax/archive/2009/03/06/brokered-authentication-for-rest-active-clients-with-saml.aspx),
I will show today how the client code can be simplified using the new
HttpClient (WCF REST Starter kit 2) and some custom http processing
stages attached to its pipeline.

The first thing we have to do is to implement a custom processing stage
(a class that derives from HttpStage) to centralize all the logic needed
to negotiate a SAML token from an existing STS.

The pipeline contains basically two kinds of stage, a regular http stage
that can be injected through the HttpClient.Stages collection, and a
more specialized implementation HttpWebRequestTransportStage, which runs
last in the pipeline and has access to all the transport settings. This
last one can only be replaced with a custom version of the HttpClient
that overrides the protected method “CreateTransportStage”,

public class HttpClient : IDisposable

{

  protected virtual HttpStage CreateTransportStage();

}

Having said this, two possible options for implementing the token
negotiation in a pipeline stage could be,

​1. A regular http stage that can be initialized with the STS address
and the user credentials through the class constructor or a property
setter.

​2. A custom HttpWebRequestTransportStage and the corresponding
HttpClient (FederatedHttpClient) implementation to return that stage.

From my point of view, the second approach seems to work better because
the HttpClient instance does not get tied to the user credentials. This
is the approach I will use for this example.

public class NegociateTokenStage : HttpWebRequestTransportStage

{

private string stsUri = "";

public NegociateTokenStage(string stsUri) : base()

{

    this.stsUri = stsUri;

}

protected override void
ProcessRequestAndTryGetResponse(HttpRequestMessage request, out
HttpResponseMessage response, out object state)

{

    string token = GetToken(stsUri, request.Uri.AbsoluteUri,
this.Settings.Credentials);

    request.Headers.Add("Authorization", token);

    base.ProcessRequestAndTryGetResponse(request, out response, out
state);

}

The custom transport stage derives from the built-in transport stage
“HttpWebRequestTransportStage” and adds some custom code in the
ProcessRequestAndTryGetResponse to negotiate the SAML token from the STS
before the final service gets called (This is being done in the GetToken
method). After that, the SAML token get passed to the final service
through the authorization html header.

The custom implementation of the HttpClient application is quite simple,
only returns our custom transport stage in the CreateTransportStage
method,

public class FederatedHttpClient : HttpClient

{

    public string StsUri

    {

        get; set;

    }

    protected override HttpStage CreateTransportStage()

    {

        NegociateTokenStage stage = new
NegociateTokenStage(this.StsUri);

        stage.Settings = this.TransportSettings;

        return stage;

    }

}

Now, the client application can use our custom version of the HttpClient
for consuming the final service, only a few lines are required.

FederatedHttpClient client = new FederatedHttpClient { StsUri =
"http://localhost:7481/STS/Service.svc/Tokens" };

client.TransportSettings.Credentials = new NetworkCredential("cibrax",
"foo");

string response =
client.Get("http://localhost:7397/RestServices/Service.svc/Claims").Content.ReadAsString();

The SAML negotiation is totally transparent to the client application,
it does not even know that a SAML token exists, sweet :).

The code is available to download at [this
location](/images/legacy/FederatedRest2.zip).

UPDATE: As John Lambert from the WCF team pointed out, a custom
transport stage also needs to override the
BeginProcessRequestAndTryGetResponse and
EndProcessRequestAndTryGetResponse to support async scenarios. I will
try to update the example to override these methods any time soon.
Thanks John for the feedback!!!.

