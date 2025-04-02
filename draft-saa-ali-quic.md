---
title: "Server-Side Migration"
abbrev: "TODO - Abbreviation"
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
 - next generation
 - unicorn
 - sparkling distributed ledger
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

normative:

informative:


--- abstract

This document specifies an extension to the QUIC transport protocol (RFC 9000) to enable mid-connection server-side migration. It defines a new QUIC frame, the SERVER_ALTERNATE_ADDRESS frame, allowing servers to dynamically advertise alternate IP addresses to clients. This extension supports use cases such as privacy-preserving address rotation, censorship circumvention, and mid-connection load balancing, expanding beyond the one-time preferred address feature defined in RFC 9000.



--- middle

# Introduction

QUIC {{?QUIC=RFC9000}} supports connection migration, allowing endpoints to change addresses (IP address and Port) while preserving connection continuity. Client Migrations, wherein a client might need to perform a connection migration, are supported at any time during the connection and after the handshake. However, Server Migrations, where a client may connect to a different server address, are limited to a one-time redirection using the \texttt{preferred\_address} transport parameter sent during the handshake. There is currently no mechanism to migrate a connection to a different server address mid-connection.

This document proposes a protocol extension enabling servers to dynamically announce alternate addresses during an active connection through the \texttt{SERVER\_ALTERNATE\_ADDRESS} frame. This explicit signaling facilitates server-directed client migration to alternate server addresses, enabling use-cases like privacy-preserving address rotation, censorship circumvention, and mid-connection load balancing.

One of the design goals for this proposal is to be minimally invasive. It introduces a single new frame type and reuses existing QUIC migration mechanisms. The aim is to maintain compatibility with existing protocol implementations and ease the path toward adoption.





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
