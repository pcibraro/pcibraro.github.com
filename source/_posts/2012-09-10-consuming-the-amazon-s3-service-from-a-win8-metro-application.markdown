---
layout: post
title: "Consuming the Amazon S3 service from a Win8 Metro Application"
date: 2012-09-10
comments: true
categories: .NET
---

As many of the existing Http APIs for Cloud Services, AWS also provides
a set of different platform SDKs for hiding many of complexities present
in the APIs. While there is a platform SDK for .NET, which is open
source and available in C\#, that SDK does not work in Win8 Metro
Applications for the changes introduced in WinRT. WinRT offers a
complete different set of APIs for doing I/O operations such as doing
http calls or using cryptography for signing or encrypting data, two
aspects that are absolutely necessary for consuming AWS. All the I/O
APIs available as part of WinRT are asynchronous, and uses the TPL model
for .NET applications (HTML and JavaScript Metro applications use a
model based in promises, which is similar concept). 

In the case of S3, the http Authorization header is used for two
purposes, authenticating clients and make sure the messages were not
altered while they were in transit. For doing that, it uses a signature
or hash of the message content and some of the headers using a symmetric
key (That's just one of the available mechanisms). Windows Azure for
example also uses the same mechanism in many of its APIs.

There are three challenges that any developer working for first time in
Metro will have to face to consume S3, the new WinRT APIs, the
asynchronous nature of them and the complexity introduced for generating
the Authorization header. Having said that, I decided to write this post
with some of the gotchas I found myself trying to consume this Amazon
service.

​1. Generating the signature for the Authorization header

All the cryptography APIs in WinRT are available under
Windows.Security.Cryptography namespace. Many of operations available in
these APIs uses the concept of buffers (IBuffer) for representing a
chunk of binary data. As you will see in the example below, these
buffers are mainly generated with the use of static methods in a WinRT
class CryptographicBuffer available as part of the namespace previously
mentioned.

```csharp
private string DeriveAuthToken(string resource, string httpMethod, string timestamp)
{
   var stringToSign = string.Format("{0}\n" +
      "\n" +
      "\n" +
      "\n" +
      "x-amz-date:{1}\n" +
      "/{2}/", httpMethod, timestamp, resource);

   var algorithm = MacAlgorithmProvider.OpenAlgorithm("HMAC_SHA1");

   var keyMaterial = CryptographicBuffer.CreateFromByteArray(Encoding.UTF8.GetBytes(this.secret));

   var hmacKey = algorithm.CreateKey(keyMaterial);

   var signature = CryptographicEngine.Sign(
                hmacKey,
                CryptographicBuffer.CreateFromByteArray(Encoding.UTF8.GetBytes(stringToSign))
            );

   return CryptographicBuffer.EncodeToBase64String(signature);
}
```

The algorithm that determines the information or content you need to use
for generating the signature is very well described as part of the AWS
documentation. In this case, this method is generating a signature
required for creating a new bucket. A HmacSha1 hash is computed using a
secret or symetric key provided by AWS in the management console. \
 \
2. Sending an Http Request to the S3 service

WinRT also ships with the System.Net.Http.HttpClient that was first
introduced some months ago with ASP.NET Web API. This client provides a
rich interface on top the traditional WebHttpRequest class, and also
solves some of limitations found in this last one. There are a few
things that don't work with a raw WebHttpRequest such as setting the
Host header, which is something absolutely required for consuming S3.
Also, HttpClient is more friendly for doing unit tests, as it receives a
HttpMessageHandler as part of the constructor that can fake to emulate a
real http call. This is how the code for consuming the service with
HttpClient looks like,

```csharp
public async Task<S3Response> CreateBucket(string name, string region = null, params string[] acl)
{
   var timestamp = string.Format("{0:r}", DateTime.UtcNow);

    var auth = DeriveAuthToken(name, "PUT", timestamp);

    var request = new HttpRequestMessage(HttpMethod.Put, "http://s3.amazonaws.com/");
    request.Headers.Host = string.Format("{0}.s3.amazonaws.com", name);
    request.Headers.TryAddWithoutValidation("Authorization", "AWS " + this.key + ":" + auth);
    request.Headers.Add("x-amz-date", timestamp);

    var client = new HttpClient();
    var response = await client.SendAsync(request);

    return new S3Response
    {
         Succeed = response.StatusCode == HttpStatusCode.OK,
         Message = (response.Content != null) ? 
            await response.Content.ReadAsStringAsync() : null
     };
 }
```

You will notice a few additional things in this code. By default,
HttpClient validates the values for some well-know headers, and
Authorization is one of them. It won't allow you to set a value with ":"
on it, which is something that S3 expects. However, that's not a problem
at all, as you can skip the validation by using the
TryAddWithoutValidation method. \
Also, the code is heavily relying on the new async and await keywords to
transform all the asynchronous calls into synchronous ones. In case you
would want to unit test this code and faking the call to the real S3
service, you should have to modify it to inject a custom
HttpMessageHandler into the HttpClient. The following implementation
illustrates this concept,

In case you would want to unit test this code and faking the call to the
real S3 service, you should have to modify it to inject a custom
HttpMessageHandler into the HttpClient. The following implementation
illustrates this concept,

```csharp
public class FakeHttpMessageHandler : HttpMessageHandler
{
  HttpResponseMessage response;

  public FakeHttpMessageHandler(HttpResponseMessage response)
  {
       this.response = response;
   }

   protected override Task<HttpResponseMessage> SendAsync(HttpRequestMessage request, System.Threading.CancellationToken cancellationToken)
   {
      var tcs = new TaskCompletionSource<HttpResponseMessage>();
            
      tcs.SetResult(response);

      return tcs.Task;
    }
}
```

You can use this handler for injecting any response while you are unit
testing the code.

