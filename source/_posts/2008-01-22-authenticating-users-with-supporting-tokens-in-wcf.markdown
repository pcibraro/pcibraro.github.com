---
layout: post
title: "Authenticating users with Supporting Tokens in WCF"
date: 2008-01-22
comments: true
categories: WCF
---

### Context

A web application used by a great number of users calls a Web service by
sending messages across a network, sometimes through one or more
intermediaries. The web service needs to identify the user logged in the
web application somehow to update data or initiate a business processes.
Some of the data within the messages is considered to be sensitive in
nature so it need to be protected.

The ideal scenario here is to use message security with a username token
to identify the logged user. Identify the user with a X509 certificate
is not practical due to the high number of users.

### Problem

How do you authenticate the user without sending the password on every
web service call ?

### Forces

Any of the following conditions justifies using the solution described
in this pattern:

**Keeping the username/password in memory on the web application is not
secure**. An attacker can gain access to that sensitive data whenever it
leaves a secure area (such as a protected memory space).

**Sending a username without password:**An attacker could pose as a
legitimate sender and send falsified messages, the message recipient can
not verify that incoming messages originated from a legitimate sender.

### Solution

Use a combination of a Mutual X509 Binding with a Usernametoken (Without
password) as supporting token. The mutual X509 binding allows message
signing and encryption using X.509 certificates. This binding accesses
the client's private key, which is used to sign the message, and the
service's public key to encrypts the message. The service decrypts the
message using its private key and verifies the signature using the
public key of the client. The public key is in the client's X.509
certificate, which is included with the message.\
The service's public key can be obtained out-of-band from a X509
certificate installed on the client machine or through a negotiation
with the service.

Each token has a different purpose:

​1. A Client X509 token for the web application, it used for data origin
authentication, which enables the recipient to verify that messages have
not been tampered with in transit (data integrity) and that they
originate from the expected sender (In this case, the trusted web site).

​2. Service X509 token, it is used to encrypt and protect sensitive data
that is contained in a message

​3. Username token, it is used identify the user that originally make
the service call.

### Implementation

Fortunately, the WCF SDK comes with examples that demonstrates the
following:

-   How to pass additional security tokens to a service through
    Supporting Tokens,
    [http://msdn2.microsoft.com/en-us/library/ms751480.aspx](http://msdn2.microsoft.com/en-us/library/ms751480.aspx)
-   How to implement a custom username password validator. Since the
    username token does not contain a password, the validator should not
    authenticate anything.
    [http://msdn2.microsoft.com/en-us/library/aa354513.aspx](http://msdn2.microsoft.com/en-us/library/aa354513.aspx)


