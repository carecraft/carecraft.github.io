---
layout: post
title:  Session Description Protocol
category : protocol-model
author: Max
tags : [sdp, protocol]
---


[RFC 4566](http://datatracker.ietf.org/doc/rfc4566/)

***

# 1. Abstract

Session Description Protocol (SDP) is intended for describing multimedia sessions for the purposes of session announcement, session invitation, and other forms of multimedia session initiation.

***

# 2. Usage

## 2.1 Multicast Announcements

A common mode of usage is for a client to announce a conference session by periodically multicasting an announcement packet to a well known multicast address and port using the Session Announcement Protocol(SAP).

SAP packets are UDP packets with the following format:
```
         |--------------------|
         | SAP header         |
         |--------------------|
         | text payload       |
         |//////////
```
The header is the Session Announcement Protocol header. The text payload is an SDP session description.  The text payload should be no greater than 1 Kbyte in length. If announced by SAP, only one session announcement is permitted in a single packet.

##  2.2 Email and WWW Announcements

For both  electronic mail and the World Wide Web distribution, the use of the MIME content type "application/sdp" should be used.  This enables the automatic launching of applications for participation in the session from the WWW client or mail reader in a standard manner.

Note that announcements of multicast sessions made only via email or the WWW do not have the property that the receiver of a session announcement can necessarily receive the session because the multicast sessions may be restricted in scope, and access to the WWW server or reception of email is possible outside this scope.

##  2.3  Streaming Media

The Real Time Streaming Protocol (RTSP) [16], is an application-level protocol for control over the delivery of data with real-time properties.  RTSP provides an extensible framework to enable controlled, on-demand delivery of real-time data, such as audio and video.  An RTSP client and server negotiate an appropriate set of parameters for media delivery, partially using SDP syntax to describe those parameters.

##  2.4 Session Initiation

The Session Initiation Protocol (SIP) is an application-layer control protocol for creating, modifying, and terminating sessions such as Internet multimedia conferences, Internet telephone calls, and multimedia distribution.  The SIP messages used to create sessions carry session descriptions that allow participants to agree on a set of compatible media types.  These session descriptions are commonly formatted using SDP.  When used with SIP, the offer/answer model provides a limited framework for negotiation using SDP.




***

# 3. Specification

SDP session descriptions are entirely textual using the ISO 10646 character set in UTF-8 encoding. SDP field names and attributes names use only the US-ASCII subset of UTF-8, but textual fields and attribute values may use the full ISO 10646 character set.

An SDP session description consists of a number of lines of text of the form `<type>=<value>`. <type> is always exactly one character and is case-significant.  <value> is a structured text string whose format depends on <type>.  It also will be case-significant unless a specific field defines otherwise.  Whitespace is not permitted either side of the `=' sign. In general <value> is either a number of fields delimited by a single space character or a free format string.

Session description (Optional items are marked with a `*` )
```
         v=  (protocol version)
         o=  (originator and session identifier)
         s=  (session name)
         i=* (session information)
         u=* (URI of description)
         e=* (email address)
         p=* (phone number)
         c=* (connection information -- not required if included in
              all media)
         b=* (zero or more bandwidth information lines)
         One or more time descriptions ("t=" and "r=" lines; see below)
         z=* (time zone adjustments)
         k=* (encryption key)
         a=* (zero or more session attribute lines)
         Zero or more media descriptions

```
Time description
```
        t=  (time the session is active)
        r=* (zero or more repeat times)
```
Media description
```
        m=  (media name and transport address)
        i=* (media title)
        c=* (connection information - optional if included at session-level)
        b=* (bandwidth information)
        k=* (encryption key)
        a=* (zero or more media attribute lines)
```

## 3.1 Connection Data

```
c=<network type> <address type> <connection address>
```
* The first sub-field is the network type, which is a text string giving the type of network.  Initially "IN" is defined to have the meaning "Internet".

* The second sub-field is the address type.  This allows SDP to be used for sessions that are not IP based. Currently only IP4 and IP6 are defined.

* The third sub-field is the connection address.  Optional extra subfields may be added after the connection address depending on the value of the `<address type>` field.

For IP4 and IP6 addresses, the connection address is defined as follows:

  1. If the session is multicast, the connection address will be an IP multicast group address.  If the session is not multicast, then the connection address contains the unicast IP address of the expected data source or data relay or data sink as determined by additional attribute fields.  It is not expected that unicast addresses will be given in a session description that is communicated by a multicast announcement, though this is not prohibited.


  2. Sessions using an IPv4 multicast connection address MUST also have a time to live (TTL) value present in addition to the multicast address, while IPv6 multicast does not use TTL scoping. TTL values MUST be in the range 0-255.  The TTL for the session is appended to the address using a slash as a separator.
```
<base multicast address>[/<ttl>]/<number of addresses>
```
If the number of addresses is not given, it is assumed to be one.


## 3.2 Media Announcements
```
m=<media> <port> <proto> <fmt> ...
```

### 3.2.1 `<media>`

The first sub-field is the media type. Currently defined media are "audio", "video", "text", "application", and "message", although this list may be extended in the future.

### 3.2.2 `<port>`

The second sub-field is the transport port to which the media stream will be sent.  The meaning of the transport port depends on the network being used as specified in the relevant "c" field and on the transport protocol defined in the third sub-field.  Other ports used by the media application (such as the RTCP port) should be derived algorithmically from the base media port.

Note: For transports based on UDP, the value should be in the range 1024 to 65535 inclusive.  For RTP compliance it should be an even number.

For applications where hierarchically encoded streams are being sent to a unicast address, it may be necessary to specify multiple transport ports.
```
          m=<media> <port>/<number of ports> <transport> <fmt list>
```
In such a case, the ports used depend on the transport protocol. For RTP, only the even ports are used for data and the corresponding one-higher odd port is used for RTCP.

If multiple addresses are specified in the "c=" field and multiple ports are specified in the "m=" field, a one-to-one mapping from port to the corresponding address is implied.  For example:
```
         c=IN IP4 224.2.1.1/127/2
         m=video 49170/2 RTP/AVP 31
```
would imply that address 224.2.1.1 is used with ports 49170 and 49171, and address 224.2.1.2 is used with ports 49172 and 49173.

Note that the semantics of multiple "m=" lines using the same transport address are undefined.  


### 3.2.3 `<proto>`

The third sub-field is the transport protocol.  The transport protocol values are dependent on the address-type field in the "c=" fields.  Thus a "c=" field of IP4 defines that the transport protocol runs over IP4.  For IP4, it is normally expected that most media traffic will be carried as RTP over UDP.  The following transport protocols are preliminarily defined, but may be extended through registration of new protocols with IANA:

* udp - denotes an unspecified protocol running over UDP.

* RTP/AVP - denotes RTP used under the RTP Profile for Audio and Video Conferences with Minimal Control running over UDP.

* RTP/SAVP: denotes the Secure Real-time Transport Protocol running over UDP.

### 3.2.4 `<fmt list>`

The fourth and subsequent sub-fields are media formats. The interpretation of the media format depends on the value of the <proto> sub-field.

* If the <proto> sub-field is "RTP/AVP" or "RTP/SAVP", the <fmt> sub-fields contain RTP payload type numbers.  When a list of payload type numbers is given, this implies that all of these payload formats MAY be used in the session, but the first of these formats SHOULD be used as the default format for the session.

* If the <proto> sub-field is "udp" the <fmt> sub-fields MUST reference a media type describing the format under the "audio", "video", "text", "application", or "message" top-level media types.  The media type registration SHOULD define the packet format for use with UDP transport.

* For media using other transport protocols, the <fmt> field is protocol specific.  Rules for interpretation of the <fmt> sub-field MUST be defined when registering new protocols.
