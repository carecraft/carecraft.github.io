---
layout: post
title: Session Initiation Protocol
category : protocol-model
author: Max
tags : [sip, voip, protocol]
---


[SIP RFC3261](http://datatracker.ietf.org/doc/rfc3261/)

***

# 1. Overview

   SIP(Session Initiation Protocol ) is an application-layer control protocol that can establish, modify, and terminate multimedia sessions (conferences) such as Internet telephony calls.  SIP can also invite participants to already existing sessions, such as multicast conferences.  Media can be added to (and removed from) an existing session.  SIP transparently supports name mapping and redirection services, which supports personal mobility - users can maintain a single externally visible identifier regardless of their network location.

   SIP supports five facets of establishing and terminating multimedia communications:

1. User location: determination of the end system to be used for communication;

1. User availability: determination of the willingness of the called party to engage in communications;

1. User capabilities: determination of the media and media parameters to be used;

1. Session setup: "ringing", establishment of session parameters at both called and calling party;

1. Session management: including transfer and termination of sessions, modifying session parameters, and invoking services.          

***

# 2. SIP session example

## 2.1 Setup

The example shows the basic functions of SIP: location of an end point, signal of a desire to communicate, negotiation of session parameters to establish the session, and teardown of the session once established.

Figure 1 shows a typical example of a SIP message exchange between two users, Alice and Bob.  (Each message is labeled with the letter "F" and a number for reference by the text.)  In this case,  atlanta.com is the domain of Bob's SIP service provider, and biloxi.com is the Bob's.  

```
                     atlanta.com  . . . biloxi.com
                 .      proxy              proxy     .
               .                                       .
       Alice's  . . . . . . . . . . . . . . . . . . . .  Bob's
      softphone                                        SIP Phone
         |                |                |                |
         |    INVITE F1   |                |                |
         |--------------->|    INVITE F2   |                |
         |  100 Trying F3 |--------------->|    INVITE F4   |
         |<---------------|  100 Trying F5 |--------------->|
         |                |<-------------- | 180 Ringing F6 |
         |                | 180 Ringing F7 |<---------------|
         | 180 Ringing F8 |<---------------|     200 OK F9  |
         |<---------------|    200 OK F10  |<---------------|
         |    200 OK F11  |<---------------|                |
         |<---------------|                |                |
         |                       ACK F12                    |
         |------------------------------------------------->|
         |                   Media Session                  |
         |<================================================>|
         |                       BYE F13                    |
         |<-------------------------------------------------|
         |                     200 OK F14                   |
         |------------------------------------------------->|
         |                                                  |

         Figure 1: SIP session setup example with SIP trapezoid
```

#### F1 INVITE Alice -> atlanta.com proxy
```
INVITE sip:bob@biloxi.com SIP/2.0
Via: SIP/2.0/UDP pc33.atlanta.com;branch=z9hG4bKnashds8
Max-Forwards: 70
To: Bob <sip:bob@biloxi.com>
From: Alice <sip:alice@atlanta.com>;tag=1928301774
Call-ID: a84b4c76e66710
CSeq: 314159 INVITE
Contact: <sip:alice@pc33.atlanta.com>
Content-Type: application/sdp
Content-Length: 142

(Alice's SDP not shown)
```

#### F2 100 Trying atlanta.com proxy -> Alice
```
SIP/2.0 100 Trying
Via: SIP/2.0/UDP pc33.atlanta.com;branch=z9hG4bKnashds8;received=192.0.2.1
To: Bob <sip:bob@biloxi.com>
From: Alice <sip:alice@atlanta.com>;tag=1928301774
Call-ID: a84b4c76e66710
CSeq: 314159 INVITE
Content-Length: 0
```

#### F3 INVITE atlanta.com proxy -> biloxi.com proxy
```
INVITE sip:bob@biloxi.com SIP/2.0
Via: SIP/2.0/UDP bigbox3.site3.atlanta.com;branch=z9hG4bK77ef4c2312983.1
Via: SIP/2.0/UDP pc33.atlanta.com;branch=z9hG4bKnashds8;received=192.0.2.1
Max-Forwards: 69
To: Bob <sip:bob@biloxi.com>
From: Alice <sip:alice@atlanta.com>;tag=1928301774
Call-ID: a84b4c76e66710
CSeq: 314159 INVITE
Contact: <sip:alice@pc33.atlanta.com>
Content-Type: application/sdp
Content-Length: 142

(Alice's SDP not shown)
```

#### F4 100 Trying biloxi.com proxy -> atlanta.com proxy
```
SIP/2.0 100 Trying
Via: SIP/2.0/UDP bigbox3.site3.atlanta.com;branch=z9hG4bK77ef4c2312983.1;received=192.0.2.2
Via: SIP/2.0/UDP pc33.atlanta.com;branch=z9hG4bKnashds8;received=192.0.2.1
To: Bob <sip:bob@biloxi.com>
From: Alice <sip:alice@atlanta.com>;tag=1928301774
Call-ID: a84b4c76e66710
CSeq: 314159 INVITE
Content-Length: 0
```

#### F5 INVITE biloxi.com proxy -> Bob
```
INVITE sip:bob@192.0.2.4 SIP/2.0
Via: SIP/2.0/UDP server10.biloxi.com;branch=z9hG4bK4b43c2ff8.1
Via: SIP/2.0/UDP bigbox3.site3.atlanta.com;branch=z9hG4bK77ef4c2312983.1;received=192.0.2.2
Via: SIP/2.0/UDP pc33.atlanta.com;branch=z9hG4bKnashds8;received=192.0.2.1
Max-Forwards: 68
To: Bob <sip:bob@biloxi.com>
From: Alice <sip:alice@atlanta.com>;tag=1928301774
Call-ID: a84b4c76e66710
CSeq: 314159 INVITE
Contact: <sip:alice@pc33.atlanta.com>
Content-Type: application/sdp
Content-Length: 142

(Alice's SDP not shown)
```

#### F6 180 Ringing Bob -> biloxi.com proxy
```
SIP/2.0 180 Ringing
Via: SIP/2.0/UDP server10.biloxi.com;branch=z9hG4bK4b43c2ff8.1;received=192.0.2.3
Via: SIP/2.0/UDP bigbox3.site3.atlanta.com;branch=z9hG4bK77ef4c2312983.1;received=192.0.2.2
Via: SIP/2.0/UDP pc33.atlanta.com;branch=z9hG4bKnashds8;received=192.0.2.1
To: Bob <sip:bob@biloxi.com>;tag=a6c85cf
From: Alice <sip:alice@atlanta.com>;tag=1928301774
Call-ID: a84b4c76e66710
Contact: <sip:bob@192.0.2.4>
CSeq: 314159 INVITE
Content-Length: 0
```

#### F7 180 Ringing biloxi.com proxy -> atlanta.com proxy
```
SIP/2.0 180 Ringing
Via: SIP/2.0/UDP bigbox3.site3.atlanta.com;branch=z9hG4bK77ef4c2312983.1;received=192.0.2.2
Via: SIP/2.0/UDP pc33.atlanta.com;branch=z9hG4bKnashds8;received=192.0.2.1
To: Bob <sip:bob@biloxi.com>;tag=a6c85cf
From: Alice <sip:alice@atlanta.com>;tag=1928301774
Call-ID: a84b4c76e66710
Contact: <sip:bob@192.0.2.4>
CSeq: 314159 INVITE
Content-Length: 0
```

#### F8 180 Ringing atlanta.com proxy -> Alice
```
SIP/2.0 180 Ringing
Via: SIP/2.0/UDP pc33.atlanta.com;branch=z9hG4bKnashds8;received=192.0.2.1
To: Bob <sip:bob@biloxi.com>;tag=a6c85cf
From: Alice <sip:alice@atlanta.com>;tag=1928301774
Call-ID: a84b4c76e66710
Contact: <sip:bob@192.0.2.4>
CSeq: 314159 INVITE
Content-Length: 0
```

#### F9 200 OK Bob -> biloxi.com proxy
```
SIP/2.0 200 OK
Via: SIP/2.0/UDP server10.biloxi.com;branch=z9hG4bK4b43c2ff8.1;received=192.0.2.3
Via: SIP/2.0/UDP bigbox3.site3.atlanta.com;branch=z9hG4bK77ef4c2312983.1;received=192.0.2.2
Via: SIP/2.0/UDP pc33.atlanta.com;branch=z9hG4bKnashds8;received=192.0.2.1
To: Bob <sip:bob@biloxi.com>;tag=a6c85cf
From: Alice <sip:alice@atlanta.com>;tag=1928301774
Call-ID: a84b4c76e66710
CSeq: 314159 INVITE
Contact: <sip:bob@192.0.2.4>
Content-Type: application/sdp
Content-Length: 131

(Bob's SDP not shown)
```

#### F10 200 OK biloxi.com proxy -> atlanta.com proxy
```
SIP/2.0 200 OK
Via: SIP/2.0/UDP bigbox3.site3.atlanta.com;branch=z9hG4bK77ef4c2312983.1;received=192.0.2.2
Via: SIP/2.0/UDP pc33.atlanta.com;branch=z9hG4bKnashds8;received=192.0.2.1
To: Bob <sip:bob@biloxi.com>;tag=a6c85cf
From: Alice <sip:alice@atlanta.com>;tag=1928301774
Call-ID: a84b4c76e66710
CSeq: 314159 INVITE
Contact: <sip:bob@192.0.2.4>
Content-Type: application/sdp
Content-Length: 131

(Bob's SDP not shown)
```

#### F11 200 OK atlanta.com proxy -> Alice
```
SIP/2.0 200 OK
Via: SIP/2.0/UDP pc33.atlanta.com;branch=z9hG4bKnashds8;received=192.0.2.1
To: Bob <sip:bob@biloxi.com>;tag=a6c85cf
From: Alice <sip:alice@atlanta.com>;tag=1928301774
Call-ID: a84b4c76e66710
CSeq: 314159 INVITE
Contact: <sip:bob@192.0.2.4>
Content-Type: application/sdp
Content-Length: 131

(Bob's SDP not shown)
```

#### F12 ACK Alice -> Bob
```
ACK sip:bob@192.0.2.4 SIP/2.0
Via: SIP/2.0/UDP pc33.atlanta.com;branch=z9hG4bKnashds9
Max-Forwards: 70
To: Bob <sip:bob@biloxi.com>;tag=a6c85cf
From: Alice <sip:alice@atlanta.com>;tag=1928301774
Call-ID: a84b4c76e66710
CSeq: 314159 ACK
Content-Length: 0
```

#### Note

The media session between Alice and Bob is now established.

Bob hangs up first.  Note that Bob's SIP phone maintains its own CSeq numbering space, which, in this example, begins with 231.  Since Bob is making the request, the To and From URIs and tags have been swapped.

#### F13 BYE Bob -> Alice
```
BYE sip:alice@pc33.atlanta.com SIP/2.0
Via: SIP/2.0/UDP 192.0.2.4;branch=z9hG4bKnashds10
Max-Forwards: 70
From: Bob <sip:bob@biloxi.com>;tag=a6c85cf
To: Alice <sip:alice@atlanta.com>;tag=1928301774
Call-ID: a84b4c76e66710
CSeq: 231 BYE
Content-Length: 0
```

#### F14 200 OK Alice -> Bob
```
SIP/2.0 200 OK
Via: SIP/2.0/UDP 192.0.2.4;branch=z9hG4bKnashds10
From: Bob <sip:bob@biloxi.com>;tag=a6c85cf
To: Alice <sip:alice@atlanta.com>;tag=1928301774
Call-ID: a84b4c76e66710
CSeq: 231 BYE
Content-Length: 0
```

There are three Via header field values - one added by Alice's SIP phone, one added by the atlanta.com proxy, and one added by the biloxi.com proxy.

In general, the end-to-end media packets take a different path from the SIP signaling messages.

## 2.2 Registration

Bob registers on start-up.  The message flow is shown in Figure 2. Note that the authentication usually required for registration is not shown for simplicity.
```
                  biloxi.com         Bob's
                   registrar       softphone
                      |                |
                      |   REGISTER F1  |
                      |<---------------|
                      |    200 OK F2   |
                      |--------------->|

                  Figure 2: SIP Registration Example
```

#### F1 REGISTER Bob -> Registrar
```
       REGISTER sip:registrar.biloxi.com SIP/2.0
       Via: SIP/2.0/UDP bobspc.biloxi.com:5060;branch=z9hG4bKnashds7
       Max-Forwards: 70
       To: Bob <sip:bob@biloxi.com>
       From: Bob <sip:bob@biloxi.com>;tag=456248
       Call-ID: 843817637684230@998sdasdh09
       CSeq: 1826 REGISTER
       Contact: <sip:bob@192.0.2.4>
       Expires: 7200
       Content-Length: 0
```
The registration expires after two hours.  The registrar responds with a 200 OK:

#### F2 200 OK Registrar -> Bob
```
       SIP/2.0 200 OK
       Via: SIP/2.0/UDP bobspc.biloxi.com:5060;branch=z9hG4bKnashds7;received=192.0.2.4
       To: Bob <sip:bob@biloxi.com>;tag=2493k59kd
       From: Bob <sip:bob@biloxi.com>;tag=456248
       Call-ID: 843817637684230@998sdasdh09
       CSeq: 1826 REGISTER
       Contact: <sip:bob@192.0.2.4>
       Expires: 7200
       Content-Length: 0
```

***

# 3. Structure of the Protocol

SIP is structured as a layered protocol, which means that its behavior is described in terms of a set of fairly independent processing stages with only a loose coupling between each stage.  

The lowest layer of SIP is its syntax and encoding.  Its encoding is specified using an augmented Backus-Naur Form grammar (BNF).

The second layer is the transport layer.  It defines how a client sends requests and receives responses and how a server receives requests and sends responses over the network.  All SIP elements contain a transport layer.

The third layer is the transaction layer.  Transactions are a fundamental component of SIP.  A transaction is a request sent by a client transaction (using the transport layer) to a server transaction, along with all responses to that request sent from the server transaction back to the client.  The transaction layer handles application-layer retransmissions, matching of responses to requests, and application-layer timeouts.  Any task that a user agent client (UAC) accomplishes takes place using a series of transactions. User agents contain a transaction layer, as do stateful proxies.  Stateless proxies do not contain a transaction layer.  The transaction layer has a client component (referred to as a client transaction) and a server component (referred to as a server transaction), each of which are represented by a finite state machine that is constructed to process a particular request.

The layer above the transaction layer is called the transaction user (TU).  Each of the SIP entities, except the stateless proxy, is a transaction user.  When a TU wishes to send a request, it creates a client transaction instance and passes it the request along with the destination IP address, port, and transport to which to send the request.  A TU that creates a client transaction can also cancel it. When a client cancels a transaction, it requests that the server stop further processing, revert to the state that existed before the transaction was initiated, and generate a specific error response to that transaction.  This is done with a CANCEL request, which constitutes its own transaction, but references the transaction to be cancelled.

***

# 4. SIP Messages

SIP messages consist of a start-line, one or more header fields, an empty line indicating the end of the header fields, and an optional message-body.
```
         generic-message  =  start-line
                             *message-header
                             CRLF
                             [ message-body ]
         start-line       =  Request-Line / Status-Line
```
   The start-line, each message-header line, and the empty line MUST be terminated by a carriage-return line-feed sequence (CRLF).  Note that the empty line MUST be present even if the message-body is not.

## 4.1 Requests

SIP requests are distinguished by having a Request-Line for a start-line.  A Request-Line contains a method name, a Request-URI, and the protocol version separated by a single space (SP) character.

The Request-Line ends with CRLF.  No CR or LF are allowed except in the end-of-line CRLF sequence.  No linear whitespace (LWS) is allowed in any of the elements.

```
Request-Line  =  Method SP Request-URI SP SIP-Version CRLF
```

1. Method:

   [RFC3261](http://datatracker.ietf.org/doc/rfc3261/) defines six methods: **REGISTER** for registering contact information, **INVITE**, **ACK**, and **CANCEL** for setting up sessions, **BYE** for terminating sessions, and **OPTIONS** for querying servers about their capabilities.  SIP extensions, documented in standards track RFCs, may define additional methods.

   Once a dialog is established, either early or confirmed, the caller and the callee can generate an
   **UPDATE** method that contains an SDP offer for the purposes of updating the session. This method is discribed in [RFC3311](https://tools.ietf.org/html/rfc3311).

   [RFC3428](https://tools.ietf.org/html/rfc3428) describes the **MESSAGE** method for sending
   instant messages using a metaphor similar to that of a two-way pager or SMS enabled handset, supporting all the requirements in [RFC2779](https://tools.ietf.org/html/rfc2779) relevant to its scope of applicability for presence and instant messaging protocols.

   The **SUBSCRIBE** method, defined in [RFC6665](https://tools.ietf.org/html/rfc6665), is used to request current state and state updates from a remote node. Then, **NOTIFY** requests are sent to inform subscribers of changes in state to which the subscriber has a subscription.

   The **REFER** method, defined in [RFC3515](https://tools.ietf.org/html/rfc3515), could also create a subscription, indicating that the recipient (identified by the Request-URI) should contact a third party using the contact information provided in the request.

   [RFC6086](https://tools.ietf.org/html/rfc6086) defines a method, **INFO**, to carry application level information between endpoints, using the SIP dialog signaling path. Note that the INFO method is not used to update characteristics of a SIP dialog or session, but to allow the applications that use the SIP
   session to exchange information (which might update the state of those applications).



2. Request-URI:

   The Request-URI is a SIP or SIPS URI or a general URI (RFC 2396).  It indicates the user or service to which this request is being addressed. The Request-URI MUST NOT contain unescaped spaces or control characters and MUST NOT be enclosed in "<>". SIP elements MAY support Request-URIs with schemes other than "sip" and "sips", for example the "tel" URI scheme of RFC 2806 . SIP elements MAY translate non-SIP URIs using any mechanism at their disposal, resulting in SIP URI, SIPS URI, or some other scheme.

3. SIP-Version:

   Both request and response messages include the version of SIP in use, and follow HTTP (with HTTP replaced by SIP, and HTTP/1.1 replaced by SIP/2.0) regarding version ordering, compliance requirements, and upgrading of version numbers.  To be compliant with this specification, applications sending SIP messages MUST include a SIP-Version of "SIP/2.0".  The SIP-Version string is case-insensitive, but implementations MUST send upper-case.

## 4.2 Responses

SIP responses are distinguished from requests by having a Status-Line as their start-line.  A Status-Line consists of the protocol version followed by a numeric Status-Code and its associated textual phrase, with each element separated by a single SP character.

No CR or LF is allowed except in the final CRLF sequence.
```
      Status-Line  =  SIP-Version SP Status-Code SP Reason-Phrase CRLF
```

***

# 5. Header Fields

Specifically, any SIP header whose grammar is of the form
```
      header  =  "header-name" HCOLON header-value *(COMMA header-value)
```
allows for combining header fields of the same name into a comma-separated list.  The Contact header field allows a comma-separated list unless the header field value is "*".

## 5.1 Header Field Format
```
         field-name: field-value
```
The format of a header field-value is defined per header-name.  It will always be either an opaque sequence of TEXT-UTF8 octets, or a combination of whitespace, tokens, separators, and quoted strings. Many existing header fields will adhere to the general form of a value followed by a semi-colon separated sequence of parameter-name, parameter-value pairs:
```
         field-name: field-value *(;parameter-name=parameter-value)
```

## 5.2 Summary of header fields

The "where" column describes the request and response types in which the header field can be used.  Values in this column are:

   * R: header field may only appear in requests;

   * r: header field may only appear in responses;

   * 2xx, 4xx, etc.: A numerical value or range indicates response codes with which the header field can be used;

   * c: header field is copied from the request to the response.

   * An empty entry in the "where" column indicates that the header field may be present in all requests and responses.

The "proxy" column describes the operations a proxy may perform on a header field:

   * a: A proxy can add or concatenate the header field if not present.

   * m: A proxy can modify an existing header field value.

   * d: A proxy can delete a header field value.

   * r: A proxy must be able to read the header field, and thus this header field cannot be encrypted.

The next six columns relate to the presence of a header field in a method:

   * c: Conditional; requirements on the header field depend on the context of the message.

   * m: The header field is mandatory.

   * m*: The header field SHOULD be sent, but clients/servers need to be prepared to receive messages without that header field.

   * o: The header field is optional.

   * t: The header field SHOULD be sent, but clients/servers need to be prepared to receive messages without that header field. If a stream-based protocol (such as TCP) is used as a transport, then the header field MUST be sent.

   * *: The header field is required if the message body is not empty.

   * -: The header field is not applicable.

Header field | Where | Proxy | ACK | BYE | CAN | INV | OPT | REG
-------------|-------|-------|-----|-----|-----|-----|-----|----
Accept | R |  |  - | o | - | o | m* | o
Accept | 2xx |  | - | - | - | o | m* | o
Accept | 415 |  | - | c | - | c | c | c
Accept-EncodingR | | | - | o | - | o | o | o
Accept-Encoding | 2xx | | - | - | - | o | m* | o
Accept-Encoding | 415 | | - | c | - | c | c | c
Accept-LanguageR | | | - | o | - | o | o | o
Accept-Language | 2xx | | - | - | - | o | m* | o
Accept-Language | 415 | | - | c | - | c | c | c
Alert-InfoRAr | |  | - | - | - | o | - | -
Alert-Info | 180 |  ar |  - | - | - | o | - | -
Allow | R | | - | o | - | o | o | o
Allow | 2xx | | - | o | - | m* | m* | o
Allow | r | |  - | o | - | o | o | o
Allow | 405 | | - | m | - | m | m | m
Authentication-Info |  2xx | | - | o | - | o | o | o
Authorization | R | |  o | o | o | o | o | o
Call-ID |  c | r |  m | m | m | m | m | m
Call-Info | | ar |  - | - | - | o | o | o
Contact |  R | |  o | - | - | m | o | o
Contact | 1xx | | - | - | - | o | - | -
Contact | 2xx | | - | - | - | m | o | o
Contact | 3xx | d |  - | o | - | o | o | o
Contact | 485 | | - | o | - | o | o | o
Content-Disposition |  | o | o | - | o | o | o
Content-Encoding | | o | o | - | o | o | o
Content-Language |  o | o | - | o | o | o
Content-Length | | ar | t | t | t | t | t | t
Content-Type  |  |  | * | * | - | * | * | *
CSeq | c | r | m | m | m | m | m | m
Date |   | a | o | o | o | o | o | o
Error-Info| 300-699 | a |  - | o | o | o | o | o
Expires  |  |  | - | - | - | o | - | o
From | c | r |  m | m | m | m | m | m
In-Reply-To | R | | - | - | - | o | - | -
Max-Forwards | R | amr | m | m | m | m | m | m
Min-Expires  |  423 | | - | - | - | - | - | m
MIME-Version | |  |  o | o | - | o | o | o
Organization |  | ar |  - | - | - | o | o | o
Priority | R | ar |  - | - | - | o | - | -
Proxy-Authenticate | 407 | ar |  - | m | - | m | m | m
Proxy-Authenticate | 401 | ar |  - | o | o | o | o | o
Proxy-Authorization | R |  dr |  o | o | - | o | o | o
Proxy-Require |  R | ar |  - | o | - | o | o | o
Record-Route | R | ar |  o | o | o | o | o | -
Record-Route |   2xx,18x | mr |  - | o | o | o | o | -
Reply-To |  |  | - | - | - | o | - | -
Require  |  | ar |  - | c | - | c | c | c
Retry-After |  404,413,480,486 |  | - | o | o | o | o | o
 | 500,503 | | - | o | o | o | o | o
 | 600,603 | | - | o | o | o | o | o
Route | R | adr | c | c | c | c | c | c
Server | r |  | - | o | o | o | o | o
Subject | R |  | - | - | - | o | - | -
Supported | R |  | - | o | o | m* | o | o
Supported | 2xx |  | - | o | o | m* | m* | o
Timestamp |   |   | o | o | o | o | o | o
To | c(1) |  r |  m | m | m | m | m | m
Unsupported | 420 |  | - | m | - | m | m | m
User-Agent  |  |  | o | o | o | o | o | o
Via | R | amr | m | m | m | m | m | m
Via | rc | dr |  m | m | m | m | m | m
Warning | r |  | - | o | o | o | o | o
WWW-Authenticate | 401 | ar |  - | m | - | m | m | m
WWW-Authenticate | 407 | ar |  - | o | - | o | o | o


***

# 6. Response Codes

## 6.1 Provisional 1xx

Provisional responses, also known as informational responses, indicate that the server contacted is performing some further action and does not yet have a definitive response.  A server sends a 1xx response if it expects to take more than 200 ms to obtain a final response.  Note that 1xx responses are not transmitted reliably. They never cause the client to send an ACK.  Provisional (1xx) responses MAY contain message bodies, including session descriptions.

Response Codes | Response Description
---------------|----------------------
100 | Trying
180 | Ringing
181 | Call Is Being Forwarded
182 | Queued
183 | Session Progress

## 6.2 Successful 2xx

Response Codes | Response Description
---------------|----------------------
200 | OK

## 6.3 Redirection 3xx

3xx responses give information about the user's new location, or about alternative services that might be able to satisfy the call.

Response Codes | Response Description
---------------|----------------------
300 | Multiple Choices
301 | Moved Permanently
302 | Moved Temporarily
305 | Use Proxy
380 | Alternative Service

## 6.4 Request Failure 4xx

Response Codes | Response Description
---------------|----------------------
400 | Bad Request
401 | Unauthorized
402 | Payment Required
403 | Forbidden
404 | Not Found
405 | Method Not Allowed
406 | Not Acceptable
407 | Proxy Authentication Required
408 | Request Timeout
410 | Gone
413 | Request Entity Too Large
414 | Request-URI Too Long
415 | Unsupported Media Type
416 | Unsupported URI Scheme
420 | Bad Extension
421 | Extension Required
423 | Interval Too Brief
480 | Temporarily Unavailable
481 | Call/Transaction Does Not Exist
482 | Loop Detected
483 | Too Many Hops
484 | Address Incomplete
485 | Ambiguous
486 | Busy Here
487 | Request Terminated
488 | Not Acceptable Here
491 | Request Pending
493 | Undecipherable

## 6.5 Server Failure 5xx

Response Codes | Response Description
---------------|----------------------
500 | Server Internal Error
501 | Not Implemented
502 | Bad Gateway
503 | Service Unavailable
504 | Server Time-out
505 | Version Not Supported
513 | Message Too Large

## 6.6 Global Failures 6xx

6xx responses indicate that a server has definitive information about a particular user, not just the particular instance indicated in the Request-URI.

Response Codes | Response Description
---------------|----------------------
600 | Busy Everywhere
603 | Decline
604 | Does Not Exist Anywhere
606 | Not Acceptable
