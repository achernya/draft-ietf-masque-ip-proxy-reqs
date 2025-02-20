--
title: Requirements for a MASQUE Protocol to Proxy IP Traffic
abbrev: IP Proxying Requirements
docname: draft-ietf-masque-ip-proxy-reqs-latest
category: info

ipr: trust200902
keyword: Internet-Draft

stand_alone: yes
pi: [toc, sortrefs, symrefs]

author:
 -
    ins: "A. Chernyakhovsky"
    name: "Alex Chernyakhovsky"
    organization: "Google LLC"
    street: "1600 Amphitheatre Parkway"
    city: "Mountain View, California 94043"
    country: "United States of America"
    email: achernya@google.com
 -
    ins: "D. McCall"
    name: "Dallas McCall"
    organization: "Google LLC"
    street: "1600 Amphitheatre Parkway"
    city: "Mountain View, California 94043"
    country: "United States of America"
    email: dallasmccall@google.com
 -
    ins: "D. Schinazi"
    name: "David Schinazi"
    organization: "Google LLC"
    street: "1600 Amphitheatre Parkway"
    city: "Mountain View, California 94043"
    country: "United States of America"
    email: dschinazi.ietf@gmail.com


--- abstract
There is interest among MASQUE working group participants in designing a
protocol that can proxy IP traffic over HTTP. This document describes the set
of requirements for such a protocol.

Discussion of this work is encouraged to happen on the MASQUE IETF mailing
list [](masque@ietf.org) or on the GitHub repository which contains the draft:
[](https://github.com/ietf-wg-masque/draft-ietf-masque-ip-proxy-reqs).


--- middle

# Introduction
There exist several IETF standards for proxying IP in a way that is
authenticated and confidential, such as IKEv2/IPsec {{?IKEV2=RFC7296}}.
However, those are distinguishable from common Internet traffic and often
blocked. Additionally, large server deployments have expressed interest in
using a VPN solution that leverages existing security protocols such as QUIC
{{!QUIC=I-D.ietf-quic-transport}} or TLS {{!TLS=RFC8446}} to avoid adding
another protocol to their security posture.

This document describes the set of requirements for a protocol that can proxy
IP traffic over HTTP. The requirements outlined below are similar to the
considerations made in designing the CONNECT-UDP method
{{?CONNECT-UDP=I-D.ietf-masque-connect-udp}}, additionally including
IP-specific requirements, such as a means of negotiating the routes that should
be advertised on either end of the connection.

Discussion of this work is encouraged to happen on the MASQUE IETF mailing list
[](masque@ietf.org) or on the GitHub repository which contains the draft:
[](https://github.com/ietf-wg-masque/draft-ietf-masque-ip-proxy-reqs).

## Conventions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this
document are to be interpreted as described in BCP 14 {{!RFC2119}} {{!RFC8174}}
when, and only when, they appear in all capitals, as shown here.

## Definitions

* Data Transport: The mechanism responsible for transmitting IP packets over
  HTTP. This can involve streams or datagrams.

* IP Session: An association between client and server whereby both agree to
  proxy IP traffic given certain configuration properties. This is similar to a
  Child Security Association in IKEv2 terminology. An IP Session uses Data
  Transports to transmit packets.

# Use Cases

There are multiple reasons to deploy an IP proxying protocol. This section
discusses some examples of use cases that MUST be supported by the protocol.
Note that while the protocol needs to support these use cases, the protocol
elements that allow them may be optional.

## Consumer VPN

Consumer VPNs refer to network applications that allow a user to hide some
properties of their traffic from some network observers. In particular, it can
hide the identity of servers the client is connecting to from the client's
network provider, and can hide the client's IP address (and derived geographical
information) from the servers they are communicating with. Note that this hidden
information is now available to the VPN service provider, so is only beneficial
for clients who trust the VPN service provider more than other entities.

## Point to Point Connectivity

Point-to-point connectivity creates a private, encrypted and authenticated
network between two IP addresses. This is useful, for example, with container
networking to provide a virtual (overlay) network with addressing separate from
the physical transport. An example of this is Wireguard.

## Point to Network Connectivity

Point-to-Network connectivity is the more traditional remote-access "VPN" use
case, frequently used when a user needs to connect to a different network (such
as an enterprise network) for access to resources that are not exposed to the
public Internet.

## Network to Network Connectivity

Network-to-Network connectivity is also called a site-to-site VPN. Similar to
the point-to-network use case, the goal is to connect two networks that are not
exposed publicly. The site-to-site aspects make this transparent to the user;
the entire networks are connected to each other and route packets transparently
without a VPN client installed on the user's device. This style of connectivity
can also be used to connect devices that cannot run VPN clients through to the
network.

# Requirements

This section lists requirements for a protocol that can proxy IP over an HTTP
connection.

## IP Session Establishment

The protocol will allow the client to request establishment of an IP Session,
along with configuration options and one or more associated Data Transports.
The server will have the ability to accept or deny the client's request.

## Proxying of IP packets

The protocol will establish Data Transports, which will be able to forward IP
packets. The Data Transports MUST be able to forward packets in their unmodified
entirety, although extensions may enable the use of modified packet formats
(e.g., compression). The protocol will support both IPv6 {{!IPV6=RFC8200}} and
IPv4 {{!IPV4=RFC0791}}.

## Maximum Transmission Unit

The protocol will allow endpoints to inform each other of the Maximum
Transmission Unit (MTU) they are willing to forward. This will allow avoiding
IP fragmentation, especially as IPv6 does not allow IP fragmentation by nodes
along the path.

## IP Assignment

The client will be able to request to be assigned an IP address range,
optionally specifying a preferred range. In response to that request, the server
will either assign a range of its choosing to the client, or decline the
request. For symmetry, the server may request assignment of an IP address range
from the client, and the client will either assign a range or decline the
request.

## Route Negotiation

At any point in an IP Session (not limited to its initial negotiation), the
protocol will allow both client and server to inform its peer that it can route
a set of IP prefixes. Both endpoints can also request a route to a given prefix,
and the peer can choose to provide that route or not.

Note that if an endpoint provides its peer with a route, the peer is in no way
obligated to route its traffic through the endpoint.

## Identity

When negotiating the creation of an IP Session, the protocol will allow both
endpoints to exchange an identifier. As examples, the identity could be a user
name, an email address, a token, or a fully-qualified domain name. Note that
this requirement does not cover authenticating the identifier.

## Transport Security

The protocol MUST be run over a protocol that provides mutual authentication,
confidentiality and integrity. Using QUIC or TLS would meet this requirement.

## Flow Control

The protocol will allow the ability to proxy IP packets without flow control,
at least when HTTP/3 is in use. QUIC DATAGRAM frames are not flow controlled
and would meet this requirement. The document defining the protocol will
provide guidance on how best to use flow control to improve IP Session
performance.

## Indistinguishability

A passive network observer not participating in the encrypted connection should
not be able to distinguish IP proxying from regular encrypted HTTP Web traffic
by only observing non-encrypted parts of the traffic. Specifically, any data
sent unencrypted (such as headers, or parts of the handshake) should look like
the same unencrypted data that would be present for Web traffic. Traffic
analysis is out of scope for this requirement.

## Support HTTP/2 and HTTP/3

The IP proxying protocol discussed in this document will run over HTTP. The
protocol SHOULD strongly prefer to use HTTP/3 {{!H3=I-D.ietf-quic-http}} and
SHOULD use the QUIC DATAGRAM frames {{!DGRAM=I-D.ietf-quic-datagram}} when
available to improve performance. The protocol SHOULD also support HTTP/2
{{!H2=RFC7540}} as a fallback when UDP is blocked on the network path. Proxying
IP over HTTP/2 MAY result in lower performance than over HTTP/3.

## Multiplexing

Since recent HTTP versions support concurrently running multiple requests over
the same connection, the protocol SHOULD support multiple independent instances
of IP proxying over a given HTTP connection.

# Extensibility

The protocol will provide a mechanism by which clients and servers can add
extension information to the exchange that establishes the IP Session. If the
solution uses an HTTP request and response, this could be accomplished using
HTTP headers.

Once the IP Session is established, the protocol will provide a mechanism that
allows reliably exchanging extension messages in both directions at any point
in the lifetime of the IP Session.

The subsections below list possible extensions that designers of the protocol
will keep in mind to ensure it will be possible to design such extensions.

## Load balancing

This extension would allow for load balancing of the traffic sent across the IP
Session, such as to another server. This allows the IP proxying mechanisms to
scale-out to multiple servers. This improves the throughput of the IP Session
beyond the limitations of a single server (and possibly in some implementations,
a single CPU). This also es the IP Session available across server
interruptions, such as routine maintenance and software upgrades.

## Authentication

Since the protocol will offer a way to convey identity, extensions will allow
authenticating that identity, from both the client and server, during the
establishment of the IP Session. For example, an extension could allow a client
to offer an OAuth Access Token {{?OAUTH=RFC6749}} when requesting an IP
Session. As another example, another extension could allow an endpoint to
demonstrate knowledge of a cryptographic secret.

## Reliable Transmission of IP Packets

While it is desirable to transmit IP packets unreliably in most cases, an
extension could provide a mechanism to allow forwarding some packets reliably.
For example, when using HTTP/3, this can be accomplished by allowing Data
Transports to run over both DATAGRAM and STREAM frames.

## Configuration of Congestion and Flow Control

An extension will allow exchanging congestion and flow control parameters to
improve performance. For example, an extension could disable congestion control
for non-retransmitted Data Transports if it knows that the proxied traffic is
itself congestion-controlled.

## Data Transport Compression

While the core protocol Data Transports will transmit IP packets in their
unmodified entirety, an extension can allow compressing these packets.

# Non-requirements

This section discusses topics that are explicitly out of scope for the IP
Proxying protocol. These topics MAY be handled by implementers or future
extensions.

## Addressing Architecture

This document only describes the requirements for a protocol that allows IP
proxying. It does not discuss how the IPs assigned are determined, managed, or
translated. While these details are important for producing a functional
system, they do not need to be handled by the protocol beyond the ability to
convey those assignments.

Similarly, "ownership" of an IP range is out of scope. If an endpoint
communicates to its peer that it can allocate addresses from a range, or route
traffic to a range, the peer has no obligation to trust that information.
Whether or not to trust this information is left to individual implementations
and deployments.

## Translation

Some servers may wish to perform Network Address Translation (NAT) or any other
modification to packets they forward. Doing so is out of scope for the proxying
protocol. In particular, the ability to discover the presence of a NAT,
negotiate NAT bindings, or check connectivity through a NAT is explicitly out
of scope and left to future extensions.

Servers that do not perform NAT will commonly forward packets similarly to how
a traditional IP router would, but the specific of that are considered out of
scope. In particular, decrementing the Hop Limit (or TTL) field of the IP header
is out of scope for MASQUE and expected to be performed by a router behind the
MASQUE server, or collocated with it.

## IP Packet Extraction

How packets are forwarded between the IP proxying connection and the physical
network is out of scope. For example, this can be accomplished on some
operating systems using a TUN interface. How this is done is deliberately not
specified and will be left to individual implementations.

# Security Considerations

This document only discusses requirements on a protocol that allows IP
proxying. That protocol will need to document its security considerations.

# IANA Considerations

This document requests no actions from IANA.

# Acknowledgments
{:numbered="false"}

The authors would like to thank participants of the MASQUE working group for
their feedback.
