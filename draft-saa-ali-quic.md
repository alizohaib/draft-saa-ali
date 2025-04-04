---
title: "Extending QUIC for Mid-Connection Server-Side Migration"
# abbrev: "TODO - Abbreviation"
category: info

docname: draft-saa-ali-quic-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
# area: AREA
# workgroup: WG Working Group
keyword:
 - QUIC
venue:
#  group: WG
#  type: Working Group
#  mail: WG@example.com
#  arch: https://example.com/WG
  github: "alizohaib/draft-saa-ali"
  latest: "https://alizohaib.github.io/draft-saa-ali/draft-saa-ali-quic.html"

author:
 -
    fullname: Ali Zohaib
    organization: UMass Amherst
    email: "azohaib@umass.edu"
 -
    fullname: Eric Wustrow
    organization: University of Colorado Boulder
    email: "ewust@colorado.edu"
 -  
    fullname: Amir Houmansadr
    organization: UMass Amherst
    email: "amir@cs.umass.edu"

normative:

informative:


--- abstract

This document proposes an extension to the QUIC transport protocol to enable mid-connection server-side migration. It defines a new QUIC frame, the SERVER_ALTERNATE_ADDRESS frame, allowing servers to dynamically advertise alternate addresses to clients. This extension supports use cases such as privacy-preserving address rotation, censorship circumvention, and mid-connection load balancing, expanding beyond the one-time preferred address feature defined in RFC 9000.



--- middle

# Introduction

QUIC version 1 {{?QUIC=RFC9000}} supports connection migration, allowing endpoints to change addresses (IP address and Port) while preserving connection continuity. Client Migrations, wherein a client might need to perform a connection migration, are supported at any time during the connection and after the handshake. However, Server Migrations, where a client may connect to a different server address, are limited to a one-time redirection using the preferred_address transport parameter sent during the handshake. There is currently no mechanism to migrate a connection to a different server address mid-connection.

This document proposes a protocol extension enabling servers to dynamically announce alternate addresses during an active connection through the SERVER_ALTERNATE_ADDRESS frame. This explicit signaling facilitates server-directed client migration to alternate server addresses, enabling use-cases like privacy-preserving address rotation, censorship circumvention, and mid-connection load balancing.

One of the design goals for this proposal is to be minimally invasive. It introduces a single new frame type and reuses existing QUIC migration mechanisms. The aim is to maintain compatibility with existing protocol implementations and ease the path toward adoption.

# Use Cases

RFC 9000 introduces the preferred_address transport parameter, allowing a server to suggest a single alternate address during the handshake. This feature is envisioned for cases like transitioning client connections from anycast to unicast addresses. However, it is limited to a one-time signal, offering no mechanism for server-initiated migration during the lifetime of a connection.

This extension generalizes the preferred address concept by allowing the server to dynamically advertise new addresses at any point via the SERVER_ALTERNATE_ADDRESS frame. This enhances QUIC’s flexibility and enables several practical use cases:

## Privacy Enhancement

By frequently rotating its IP addresses and signaling these changes using the new frame, a server can significantly reduce the effectiveness of traffic analysis by an adversary. Each migration to a new server address fragments the connection into multiple, seemingly independent sub-flows, potentially traversing different network paths. This fragmentation obscures the association between packets and endpoints, even when metadata from encrypted traffic is visible. As a result, traffic analysis techniques such as Website Fingerprinting become more difficult, as adversaries face increased uncertainty when attempting to probabilistically correlate and reconstruct the original flow.


## Censorship Circumvention

In restrictive environments where QUIC endpoints are subject to IP-based blocking, server migration can be a powerful evasion technique. The server can periodically shift its active connection footprint across a large IPv6 space or a pool of IPv4 addresses, sending SAA frames to inform clients in real time. This strategy can effectively outpace the censor’s ability to blacklist IP addresses (as these IP addresses can be ephemeral) and offers better adaptability than static transport-layer obfuscation methods.

MASQUE proxies can leverage this capability to provide censorship-resistant tunnels. By continuously migrating the IP addresses of upstream egress proxies, MASQUE clients can maintain long-lived QUIC connections through transient network paths that evade IP-based filtering.


## Mid-Connection Load Balancing

The ability to signal alternate addresses mid-connection also enables fine-grained load balancing strategies. A server that detects impending overload or hardware failure can instruct connected clients to migrate to a different server node within the same cluster. This migration can happen without interrupting the connection, avoiding the need for upper-layer renegotiation or re-authentication. Unlike DNS-based load balancing, this approach operates directly within the transport layer, preserving connection continuity and transport-level performance optimizations.

## Graceful Failover and Maintenance

In cloud and data center environments, servers are routinely brought down for updates, scaling, or scheduled maintenance. This extension enables graceful failover by allowing a server to notify clients before termination and instruct them to migrate to a backup or replacement node. Because migration occurs at the transport level, application-layer sessions remain uninterrupted.


# Server Alternate Address Frame

SERVER_ALTERNATE_ADDRESS frame is formatted as shown in {{fig-saa-format}}.

~~~
SERVER_ALTERNATE_ADDRESS Frame {
    Type (i) = 0xTBD,
    Sequence Number (i),
    IPv4 Address (32),
    IPv4 Port (16),
    IPv6 Address (128),
    IPv6 Port (16),
    Connection ID Length (8),
    Connection ID (..),
    Stateless Reset Token (128),
}

~~~
{: #fig-saa-format title="SERVER_ALTERNATE_ADDRESS Frame Format"}


## Client Behavior

This extension introduces a new transport parameter, enable_server_migration (TYPE: TBD), which is included in the handshake to indicate the endpoint's willingness to receive Server Alternate Address (SAA) frames. This parameter has a zero-length value.

When a client receives an SAA frame, it MAY initiate migration by validating the advertised addresses. This validation is performed based on the client's current IP family and the Connection IDs included in the frame. If validation succeeds, the client migrates the connection to the new address using new Connection IDs, following the same process used for the preferred address transport parameter.


## Server Behavior

If the client includes the enable_server_migration transport parameter in the handshake, the server sends a SERVER_ALTERNATE_ADDRESS frame. Servers that do not support this feature MAY ignore the client's transport parameters.

Similar to the preferred_address transport parameter, servers MAY provide an alternate address family (e.g., IPv4 or IPv6), allowing clients to choose the one best suited to their network environment.

# Conventions and Definitions

{::boilerplate bcp14-tagged}


# Security Considerations

TODO Security


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
