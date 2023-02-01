---
title: "An Update to the tcpControlBits IP Flow Information Export (IPFIX) Information Element"
abbrev: "tcpControlBits IPFIX"
category: std

docname: draft-boucadair-opsawg-rfc7125-update-latest
submissiontype: IETF
number:
date:
consensus: true
v: 3
area: "Operations and Management"
workgroup: "Operations and Management Area Working Group"
keyword:
 - IPFIX
 - TCP
 - Measurement
 - Export
 - Observability

author:
 -
    fullname: Mohamed Boucadair
    organization: Orange
    country: France
    city: Rennes
    code: 35000
    email: mohamed.boucadair@orange.com

normative:
  TCP-FLAGS:
    title: TCP Header Flags
    author:
      org: IANA
    target: https://www.iana.org/assignments/tcp-parameters/tcp-parameters.xhtml#tcp-header-flags

informative:
  IPFIX:
    title: IP Flow Information Export (IPFIX) Entities
    author:
      org: IANA
    target: <https://www.iana.org/assignments/ipfix/ipfix.xhtml

--- abstract

   RFC 7125 revised the tcpControlBits IP Flow Information Export
   (IPFIX) Information Element that was originally defined in RFC 5102
   to reflect changes to the TCP Flags header field since RFC 793.
   However, that update is still problematic for interoperability
   because some flag values were deprecated since then.

   This document updates RFC 7125 by removing stale information from the
   IPFIX registry and avoiding future conflicts with the authoritative
   TCP Header Flags registry.

--- middle

#  Introduction

   TCP defines a set of control bits (also known as "flags") for
   managing connections.  The "Transmission Control Protocol (TCP)
   Header Flags" registry was initially set by {{?RFC3168}}, but it was
   populated with only TCP control bits that were defined in {{?RFC3168}}.
   {{!RFC9293}} fixed that by moving that registry to be listed as a
   subregistry under the "Transmission Control Protocol (TCP)
   Parameters" registry, adding bits that had previously been specified
   in {{?RFC793}}, and removing the NS (Nonce Sum) bit as per {{?RFC8311}}.
   Also, {{!RFC9293}}  introduces "Bit Offset" to ease referencing each
   header flag's offset within the 16-bit aligned view of the TCP header
   (Section 3.1 of {{!RFC9293}}).  {{TCP-FLAGS}} is thus settled as the
   authoritative reference for the assigned TCP control bits.

   {{!RFC7125}} revised the tcpControlBits IP Flow Information Export
   (IPFIX) Information Element (IE) that was originally defined in
   {{?RFC5102}} to reflect changes to the TCP Flags header field since
   {{?RFC793}}.  However, that update is still problematic for
   interoperability because a value was deprecated since then (Section 7
   of {{?RFC8311}}) and, therefore, {{!RFC7125}} risks to deviate from the
   authoritative TCP registry {{TCP-FLAGS}}.

   This document fixes that problem by removing stale information from
   the IPFIX registry and avoiding future conflicts with the
   authoritative TCP registry.

   Also, because the setting of control bits may be misused in some
   flows (e.g., DDoS attacks), an exporter has to report all observed
   control bits even if no meaning is associated with a given TCP flag.
   This document uses a stronger requirement language compared to
   {{!RFC7125}}.  See Section 3 for more details.

#  Terminology

{::boilerplate bcp14-tagged}

   This document uses the terms defined in Section 2 of {{!RFC7011}}.

#  An Update to tcpControlBits IP Flow Information Export (IPFIX)
    Information Element

   This document updates Section 3 of {{!RFC7125}} as follows:

~~~~
   OLD:
      The values of each bit are shown below, per the definition of the
      bits in the TCP header [RFC0793][RFC3168] [RFC3540]:

             MSb                                                         LSb
        0   1   2   3   4   5   6   7   8   9  10  11  12  13  14  15
      +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
      |               |           | N | C | E | U | A | P | R | S | F |
      |     Zero      |   Future  | S | W | C | R | C | S | S | Y | I |
      | (Data Offset) |    Use    |   | R | E | G | K | H | T | N | N |
      +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+

      bit    flag
      value  name  description
      ------+-----+-------------------------------------
      0x8000       Zero (see tcpHeaderLength)
      0x4000       Zero (see tcpHeaderLength)
      0x2000       Zero (see tcpHeaderLength)
      0x1000       Zero (see tcpHeaderLength)
      0x0800       Future Use
      0x0400       Future Use
      0x0200       Future Use
      0x0100   NS  ECN Nonce Sum
      0x0080  CWR  Congestion Window Reduced
      0x0040  ECE  ECN Echo
      0x0020  URG  Urgent Pointer field significant
      0x0010  ACK  Acknowledgment field significant
      0x0008  PSH  Push Function
      0x0004  RST  Reset the connection
      0x0002  SYN  Synchronize sequence numbers
      0x0001  FIN  No more data from sender

      As the most significant 4 bits of octets 12 and 13 (counting from
      zero) of the TCP header [RFC0793] are used to encode the TCP data
      offset (header length), the corresponding bits in this Information
      Element MUST be exported as zero and MUST be ignored by the
      collector.  Use the tcpHeaderLength Information Element to encode
      this value.

      Each of the 3 bits (0x800, 0x400, and 0x200), which are reserved
      for future use in [RFC0793], SHOULD be exported as observed in the
      TCP headers of the packets of this Flow.
~~~~

~~~~
   NEW:
      As per [RFC9293], the assignment of the TCP control bits is
      managed by IANA from the "TCP Header Flags" registry [TCP-FLAGS].
      That registry is authoritative to retrieve the most recent TCP
      control bits.

      As the most significant 4 bits of octets 12 and 13 (counting from
      zero) of the TCP header [RFC9293] are used to encode the TCP data
      offset (header length), the corresponding bits in this Information
      Element MUST be exported with a value of zero and MUST be ignored
      by the collector.  Use the tcpHeaderLength Information Element to
      encode this value.

      All TCP control bits (including those unassigned) MUST be exported
      as observed in the TCP headers of the packets of this Flow.
~~~~


#  IANA Considerations

   IANA is requested to update the "tcpControlBits" entry of the {{IPFIX}}
   as follows:

   *  Update the description of to reflect the change in Section 3.

   *  Add {{TCP-FLAGS}} to the Additional Information field.

   *  Add this document to the references.

# Security Considerations

   This document does not add any new security considerations to those
   already discussed in Section 5 of {{!RFC7125}}.

--- back

# Acknowledgments
{:numbered="false"}

   This document was triggered by a discussion in opswag with the
   authors of {{?I-D.ietf-opsawg-ipfix-srv6-srh}}.

   Thanks to Christian Jacquenet, Thomas Graf, and Benoît Claise for the
   review and comments.
