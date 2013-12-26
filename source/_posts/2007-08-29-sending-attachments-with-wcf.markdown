---
layout: post
title: "Sending Attachments with WCF"
date: 2007-08-29
comments: true
categories: WCF
---

WCF mainly supports two modes to send attachments in a efficient way, an
streamed mode and a buffered/chunked mode.

In the first mode, the transport basically transfers large amount of
data using small buffers with a minimal overhead. This mode of
transferring the data has some drawbacks, for instance it does not
support message security (Only transport security through SSL), or the
robustness of reliable messaging. On the other hand, interoperability is
easier to archive since no WS-\* standard is used. Nicholas describes
the restrictions very well on this post,
[http://blogs.msdn.com/drnick/archive/2006/03/31/565558.aspx](http://blogs.msdn.com/drnick/archive/2006/03/31/565558.aspx)

I will show a simple example that uses the Streamed mode to upload a
file to the server. The contract for this service looks as follow:

[ServiceContract()]

public interface IFileTransferService

{

  [OperationContract(IsOneWay = true)]

  void Upload(FileTransferRequest request);

}

As Nicholas mentioned on his post, when this mode is used, only the
stream object can go in the message body, the rest of the arguments must
go as headers. That is the main reason of using a message contract
FileTransferRequest.

[MessageContract()]

public class FileTransferRequest

{

  [MessageHeader(MustUnderstand = true)]

  public string FileName;

 

  [MessageBodyMember(Order = 1)]

  public System.IO.Stream Data;

 

}

Processing large files with buffered transfer does not scale well, and
could be something impossible to do because of the high memory
requeriments on the client and server side. The workaround for this is
chunking combined with reliable messaging, which basically split large
files in small fragments (For instance, 64 o 128 KB fragments) and use
reliable messaging to assure the ordered delivery of those fragments.
WCF does not provide a transport with chunk support by default, but
fortunately it is implemented in an example of the SDK.

This simple is under the folder [SDK
folder]\\Samples\\Extensibility\\Channels\\ChunkingChannel

In addition to the mode used to transfer the streams, WCF also supports
the concept of message encoders, which basically serializes a
xml infoset into a stream of bytes using an specific encoding. WCF ships
by default with three encoders, Text, Binary and MTOM. Any of
the features provided by these encoders should be considered carefully
to avoid performance issues in the application.

The Text and MTOM encoders provides interoperability with other
platforms while the Binary formatter uses a proprietary format. Anyway,
let's discuss these types of encoder more in detail:

**Text Encoder**: As it name says, it is a text message encoder and
supports both plain XML encoding (For Rest/POST scenarios) as well as
SOAP Encoding. When this encoder is used, the attachments are encoded as
base64 and added as part of the xml message. This transformation to a
base64 encoding makes the final size of the attachments bigger, almost
30% bigger than the original size.

**Binary Encoder**: It uses a compact binary format, optimized for a WCF
to WCF communication. This encoder is the recommend one for sending
attachments when interoperability is not a requirement.

**MTOM Encoder**:  It uses a Message Transmission Optimization Mechanism
(MTOM) encoding. MTOM is a efficient technology for transmitting large
blocks of binary data in a soap message as-is, without conversion to
text. In addition, MTOM is a interoperable standard and also supports
Message Security (WS-Security). When Message Security is used, WCF
basically performs the following steps:

​1. Encodes all the blocks of binary data as Base64.

​2. Applies message security (Integrity and Confidentiality) to the
resulting block of Base64 text.

​3. Adds the security headers to the soap message

​4. Transmits the binary data separately from the soap message (The
Base64 data created in the step 2 is discarded)

For more details about moving large amounts of data and final
recommendations, see this [excellent
post](http://blogs.msdn.com/yassers/archive/2006/01/21/515887.aspx)
of [Yasser Shohoud](http://blogs.msdn.com/yassers/).

Examples,

[Buffered Upload](/images/legacy/BufferedUpload.zip)

[Streamed Upload](/images/legacy/StreamedUpload.zip)

