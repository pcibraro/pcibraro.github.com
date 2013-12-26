---
layout: post
title: "Securing web services"
date: 2006-08-23
comments: true
categories: WCF
---

This post discusses first the most important aspects to consider
regarding security, and then, how these aspects are implemented with web
services.

### Security Aspects

Sending and receiving plain text messages through unsecured networks can
lead to attacks performed by an attacker who potentially can intercept
these messages and modify them for malicious purposes.

By using some security mechanisms, the sensitive data can be protected
against threats such as eavesdropping or data tampering.

Usually, the following security aspects are considered for any
application:

-   ***Authentication***: Is the process of identifying the user, and
    making sure that the user is who he says he is.
-   ***Authorization***: Defines the rights and permissions of users.
-   ***Message Integrity***: Guarantees that a message has not been
    changed in transit.
-   ***Message Confidentiality***: Encrypts a message so that
    unauthorized entities cannot view its content.
-   ***Non-Repudiation***: Guarantees that the sender of a message
    cannot later deny having sent the message and that the recipient
    cannot deny having received it.

### Transport security vs. Message layer security

This is worth mentioning since the difference between these two modes to
assure security between two endpoints is not generally known.

#### **Transport security**

Transport security represents an approach where the underlying transport
or application servers are used to handle security features. When the
message leaves the transport, it is no longer secure. For example,
Secure Sockets Layer (SSL) is a common transport layer approach that is
used to provide encryption.

This approach does not support multiple intermediates, if a message
needs to go through multiple endpoints to reach its destination, each
intermediate point must forward the message over a new SSL connection
(the original message from the client is not protected on each
intermediary).

 

#### **Message security**

Message layer security represents an approach where all the information
related to security is encapsulated in the message. Securing the message
using this approach instead of using transport security has several
advantages that include:

 

-   **Increased flexibility**: Parts of the message, instead of the
    entire message, can be encrypted or signed. This means that
    intermediaries can view parts of the message that are intended for
    them.
-   **Support for multiple transports**: The secured messages can be
    sent over many different transports, such as Simple Mail Transfer
    (SMTP), HTTP, Message Queues, without having to rely on the protocol
    for security. The message is still secure after taking it out from
    the underlying transport.

### ***Web Services***

The web services by themselves do not provide the quality of service
required by many business applications these days. (e.g. transactions,
security, and reliability).

Due to that reason, different vendors such as Microsoft, Oracle, IBM,
Sun and others have coordinated their efforts to improve that quality of
service, and as result of this work, they developed the WS-\* protocols.

WS-\* is the name for a set of protocols that extend the normal web
service functionality modifying the SOAP messages (They work at message
level).

For the moment, these are some of the available protocols that can be
used to provide a better quality of service:

 

-   **WS-Security**: Describes enhancements to SOAP messaging to provide
    message integrity, message confidentiality, and single message
    authentication. The specified mechanisms can be used to accommodate
    a wide variety of security models and encryption technologies.
-   **WS-Addressing:**Provides transport-neutral mechanisms to address
    web services and messages. Specifically, this specification defines
    XML elements to identify web services endpoints and to secure
    end-to-end endpoint identification in messages. In other words, it
    enables message transmission in a transport-neutral manner.
    Therefore, HTTP is no longer required as unique transport for web
    services, and other kinds of transports can be used, such as SMTP,
    TCP, and even MSMQ.
-   **MTOM:**Provides a mechanism for optimizing the transmission and/or
    wire format of a SOAP message when file attachments need to be
    included.
-   **WS-ReliableMessaging**:  Describes a protocol that allows messages
    to be delivered reliably between distributed applications in the
    presence of software components, system, or network failures. This
    protocol can be implemented using different network transport
    technologies.
-   **WS-Transaction:**Provides a mechanism to establish short-lived
    distributed transactions between applications that have
    all-or-nothing semantics.

****

Microsoft already implemented WS-Security, WS-Addressing and MTOM in the
Microsoft WSE framework, but it is planning to implement all of them in
WCF (Windows Communication Foundation). WCF is a messaging framework to
build distributed application that will be available in a couple of
months.

 

Below, a summary about the advantages of using web services.

 

**Advantages**

****

-   It is highly interoperable. Almost all the application vendors
    support basic web services and the WS-\* specifications as well.
-   It works in a transport-neutral manner, any transport can be used.
-   It is based on XML and other open standards. The application does
    not need to provide a custom mechanism to parse or transport the
    messages.
-   They can provide message authentication, integrity and
    confidentiality.


