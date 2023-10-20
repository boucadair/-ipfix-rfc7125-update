---
title: "An Update to the tcpControlBits IP Flow Information Export (IPFIX) Information Element"
abbrev: "tcpControlBits IPFIX"
category: std
obsoletes: 7125

docname: draft-ietf-opsawg-rfc7125-update-latest
submissiontype: IETF
number:
date:
consensus: true
v: 3
area: "Operations and Management"
workgroup: "OPSAWG"
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
   to reflect changes to the TCP header control bits since RFC 793.
   However, that update is still problematic for interoperability
   because some flag values have subsequently been deprecated.

   This document removes stale information from the
   IPFIX registry and avoids future conflicts with the authoritative
   TCP Header Flags registry.

   This document obsoletes RFC 7125.

--- middle

#  Introduction

   TCP defines a set of control bits (also known as "flags") for
   managing connections ({{Section 3.1 of !RFC9293}}). The "Transmission Control Protocol (TCP)
   Header Flags" registry was initially set by {{?RFC3168}}, but it was
   populated with only TCP control bits that were defined in {{?RFC3168}}.
   {{!RFC9293}} fixed that by moving that registry to be listed as a
   subregistry under the "Transmission Control Protocol (TCP)
   Parameters" registry {{TCP-FLAGS}}, adding bits that had previously been specified
   in {{?RFC0793}}, and removing the NS (Nonce Sum) bit as per {{?RFC8311}}.
   Also, {{Section 6 of !RFC9293}} introduces "Bit Offset" to ease referencing each
   header flag's offset within the 16-bit aligned view of the TCP header
   (Figure 1 of {{!RFC9293}}). {{TCP-FLAGS}} is thus settled as the
   authoritative reference for the assigned TCP control bits.

   > Note: The bits in offsets 0 through 3 are not header flags, but the TCP segment Data Offset field.

   {{?RFC7125}} revised the tcpControlBits IP Flow Information Export
   (IPFIX) Information Element (IE) that was originally defined in
   {{?RFC5102}} to reflect changes to the TCP control bits since
   {{?RFC0793}}.  However, that update is still problematic for
   interoperability because a value was deprecated since then ({{Section 7 of ?RFC8311}})
   and, therefore, {{?RFC7125}} risks to deviate from the
   authoritative TCP registry {{TCP-FLAGS}}.

   This document fixes that problem by removing stale information from
   the IPFIX registry {{IPFIX}} and avoiding future conflicts with the
   authoritative TCP registry {{TCP-FLAGS}}. The update in this document is also useful
   to enhance observability. For example, network operators can identify
   when packets are being observed with unassigned TCP flags set and,
   therefore, identify which applications in the network should be upgraded
   to reflect the changes to TCP flags that were introduced, e.g., in {{?RFC8311}}.

   The main changes to {{?RFC7125}} are listed in {{changes}}.

#  Terminology

{::boilerplate bcp14-tagged}

   This document uses the terms defined in Section 2 of {{!RFC7011}}.

#  The tcpControlBits Information Element

ElementId:
: 6

Data Type:
: unsigned16

Data Type Semantics:
: flags

Description:
: TCP control bits observed for the packets of this Flow.
  This information is encoded as a bit field; each TCP control
  bit has a corresponding bit in that field. A bit is set to 1 if any
  observed packet of this Flow has the corresponding TCP control bit
  set to 1. The bit is cleared to 0 otherwise.

: As per {{!RFC9293}}, the assignment of the TCP control bits is
  managed by IANA from the "TCP Header Flags" registry {{TCP-FLAGS}}.
  That registry is authoritative to retrieve the most recent TCP
  control bits.

: As the most significant 4 bits of octets 12 and 13 (counting from
  zero) of the TCP header {{!RFC9293}} are used to encode the TCP data
  offset (header length), the corresponding bits in this Information
  Element MUST be exported with a value of zero and MUST be ignored
  by the collector. Use the tcpHeaderLength Information Element to
  encode this value.

: All TCP control bits (including those unassigned) MUST be exported
  as observed in the TCP headers of the packets of this Flow.

: If exported as a single octet with reduced-size encoding, this
  Information Element covers the low-order octet of this field (i.e.,
  bit offset positions 8 to 15) {{TCP-FLAGS}}. A collector receiving this Information Element
  with reduced-size encoding must not assume anything about the
  content of the four bits with bit offset positions 4 to 7.

: Exporting Processes exporting this Information Element on behalf
  of a Metering Process that is not capable of observing any of the
  flags with bit offset positions 4 to 7 SHOULD use reduced-size encoding,
  and only export the least significant 8 bits of this Information
  Element.

: Note that previous revisions of this Information Element's
  definition specified that flags with bit offset positions 8 and 9 must be exported as
  zero, even if observed.  Collectors should therefore not assume
  that a value of zero for these bits in this Information Element
  indicates the bits were never set in the observed traffic,
  especially if these bits are zero in every Flow Record sent by a
  given exporter.

: Note also that {{TCP-FLAGS}} indexes the bit offset from the most-significant
  bit of octet 12 to the least-significant bit of octet 13 in the TCP header,
  but the tcpControlBits is encoded as a regular unsigned 16 bit integer.

: For example, a tcpControlBits Information Element set to 0x90 is used to report TCP control bits for a Flow
  that has CWR (Congestion Window Reduced) and ACK flag bits set (that is,
  bit offset positions 8 and 11).

~~~~
MSB                           LSB
                     1
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|0|0|0|0|0|0|0|0|1|0|0|1|0|0|0|0|
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
~~~~
{: artwork-align="center"}

Units:

Range:

References:
: {{!RFC9293}}[This-Document]

Additional Information:
: See the assigned TCP control bits in {{TCP-FLAGS}}.

Revision:
: 2


#  IANA Considerations

   IANA is requested to update the "tcpControlBits" entry of the {{IPFIX}}
   to echo the details provided in Section 3.

# Security Considerations

   Because the setting of TCP control bits may be misused in some
   flows (e.g., Distributed Denial-of-Service (DDoS) attacks), an exporter
   has to report all observed control bits even if no meaning is associated
   with a given TCP flag. This document uses a stronger requirement language
   compared to {{?RFC7125}}.

   This document does not add new security considerations to those already
   discussed for IPFIX in {{!RFC7011}}.

--- back

# Changes from RFC 7125 {#changes}

* Clean-up the description by removing mentions of stale flag bits, referring to the flag bits by their bit offset position, and relying upon the IANA TCP registry.
* Remove the table of TCP flag bits from the description of the tcpControlBits Information Element.
* Add {{TCP-FLAGS}} to the Additional Information field of the tcpControlBits Information Element.
* Use strong normative language for exporting observed flags.
* Update the references of the tcpControlBits Information Element.
* Bump the revision of the tcpControlBits Information Element.
* Replace obsolete RFCs (e.g., {{?RFC0793}}).


# Acknowledgments
{:numbered="false"}

   This document was triggered by a discussion of the author in opsawg with the
   authors of {{?I-D.ietf-opsawg-ipfix-srv6-srh}}.

   Thanks to Christian Jacquenet, Thomas Graf, and Benoît Claise for the
   review and comments.

   Thanks to Michael Scharf for the tsvart review and Ketan Talaulikar for the rtgdir review.

   Thanks to Rob Wilton for the AD review.

   Thanks for Tim Bray for the artart review and Shawn Emery for the secdir review.

  Acknowledgments from {{?RFC7125}}:
  : Thanks to Andrew Feren, Lothar Braun, Michael Scharf, and Simon
     Josefsson for comments on the revised definition.  This work is
     partially supported by the European Commission under grant agreement
     FP7-ICT-318627 mPlane; this does not imply endorsement by the
     Commission.

# Contributors
{:numbered="false"}

The authors of {{?RFC7125}} are as follows:

* Brian Trammell
* Paul Aitken

