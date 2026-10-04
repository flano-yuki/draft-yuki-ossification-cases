---
title: "Cases of Protocol Ossification on the Internet"
abbrev: "Ossification Cases"
category: info
docname: draft-yuki-ossification-cases-latest
submissiontype: independent
consensus: false
v: 3
area: Internet
workgroup: ""
keyword:
  - protocol ossification
  - middlebox
  - protocol evolution
date: 2026-09-29
author:
  - fullname:
      :: 後藤ゆき
      ascii: Yuki Goto
    organization: independent
    email: minami.hiroy@gmail.com
normative:
  RFC3168:
  RFC6891:
  RFC7045:
  RFC7413:
  RFC7821:
  RFC7822:
  RFC7872:
  RFC8027:
  RFC8041:
  RFC8446:
  RFC8684:
  RFC8701:
  RFC8841:
  RFC8906:
  RFC9000:
  RFC9001:
  RFC9065:
  RFC9098:
  RFC9170:
  RFC9288:
  RFC9369:
  RFC9769:
  RFC9849:
  RFC9868:
  RFC10001:
informative:
  EDELINE-UDP:
    display: Edeline
    title: "Using UDP for Internet Transport Evolution"
    author:
      - name: Korian Edeline
      - name: Mirja Kühlewind
      - name: Brian Trammell
      - name: Emile Aben
      - name: Benoit Donnet
    date: 2016
    target: https://arxiv.org/abs/1612.07816
    refcontent: "arXiv:1612.07816"
  PAASCH-TFO:
    display: Paasch
    title: "Deploying TCP Fast Open in the wild"
    author:
      - name: Christoph Paasch
    refcontent: "IETF 94 TCPM presentation"
    target: https://www.ietf.org/proceedings/94/slides/slides-94-tcpm-13.pdf
  BENJAMIN-TLS13:
    display: Benjamin
    title: "Additional TLS 1.3 results from Chrome"
    author:
      - name: David Benjamin
    date: 18 December 2017
    refcontent: "TLS Working Group mailing list"
    target: https://mailarchive.ietf.org/arch/msg/tls/i9blmvG2BEPf1s1OJkenHknRw9c/
  CECPQ2:
    title: CECPQ2
    author:
      - organization: Chromium
    target: https://www.chromium.org/cecpq2/
  FORTINET-MLKEM:
    title: "Technical Tip: ERR_SSL_PROTOCOL_ERROR when using Flow-based Deep Inspection due to ML-KEM post-quantum TLS key exchange (Known Issue)"
    author:
      - organization: Fortinet
    target: https://community.fortinet.com/fortigate-3/technical-tip-err-ssl-protocol-error-when-using-flow-based-deep-inspection-due-to-ml-kem-post-quantum-tls-key-exchange-known-issue-189132
  PALOALTO-PQC:
    title: "Palo Alto Networks technical note on fragmented ClientHello inspection"
    author:
      - organization: Palo Alto Networks
    target: https://knowledgebase.paloaltonetworks.com/KCSArticleDetail?id=kA14u000000TperCAC&lang=ja
  DNSOP-GREASE:
    title: "Greasing Protocol Extension Points in the DNS"
    author:
      - name: Shumon Huque
      - name: Mark P. Andrews
    target: https://datatracker.ietf.org/doc/html/draft-ietf-dnsop-grease-03
    refcontent: "Internet-Draft, draft-ietf-dnsop-grease-03"
  HTTP-GREASE:
    title: "Greasing HTTP"
    author:
      - name: Mark Nottingham
    target: https://datatracker.ietf.org/doc/draft-nottingham-http-grease/
    refcontent: "Internet-Draft, draft-nottingham-http-grease"
  LANGLEY-QUIC:
    display: Langley
    title: "The QUIC Transport Protocol: Design and Internet-Scale Deployment"
    target: https://doi.org/10.1145/3098822.3098842
    refcontent: "SIGCOMM 2017"
  QUICHE-CHAOS:
    title: "QUIC Chaos Protector implementation"
    author:
      - organization: Chromium
    target: https://quiche.googlesource.com/quiche/+/refs/heads/main/quiche/quic/core/quic_chaos_protector.cc
  NTPV5-DRAFT:
    title: "Network Time Protocol Version 5"
    author:
      - name: Miroslav Lichvar
      - name: Tal Mizrahi
    target: https://datatracker.ietf.org/doc/html/draft-ietf-ntp-ntpv5#section-12
    refcontent: "Internet-Draft, draft-ietf-ntp-ntpv5-09, Section 12"
---

--- abstract

This document catalogues cases of protocol ossification.  Protocol
ossification is a phenomenon in which a new protocol, version, or extension
cannot traverse an existing Internet path; such problems have been discovered
and reported during protocol standardization and deployment.

This document
summarizes the reported observations and sources for those cases, together with
case-specific responses.

--- middle

# Introduction

Protocol specifications can contain extension points for future versions,
options, extension fields, and other additions.  Such extension points
normally define rules intended to preserve interoperability in the presence
of unknown parameters.

However, even when an extension point is permitted by the specification,
network devices or middleware on the path can make assumptions about packet
formats and values based on what they already recognize.  A new value or
structure can violate those assumptions and disrupt communication.
This phenomenon--where deploying an extension or a new version of an existing
protocol over the Internet is impeded--is called *protocol ossification*.

Protocol ossification has been observed in a range of protocols, including
TCP, TLS 1.3, and QUIC.  This document summarizes such cases and their
mitigations.  Each mitigation distinguishes specified measures, implemented
measures, and observed deployment actions recorded by a source, as applicable.

# Terminology

Endpoint:
: A host, application, or service that initiates or terminates communication.

Middlebox:
: A device or function between endpoints that does more than forward packets,
  including state tracking, inspection, transformation, or policy enforcement.
  NATs, firewalls, proxies, and load balancers are examples.

Ossification:
: Loss of interoperability for a valid extension when an endpoint or an
  on-path implementation fails to handle an unknown value or new structure
  as required by the specification.  It might reject, drop, rewrite, or
  misinterpret a version, code point, option, extension field, or header
  field reserved or defined for future use.

Intentional blocking:
: A deliberate decision not to permit an unknown feature because of a security
  policy, regulation, or operational requirement.  It can produce the same
  connection failure as ossification, but its cause is different.  This
  document distinguishes the two.

GREASE:
: A technique that sends reserved values with no semantic meaning during normal
  operation, exposing implementations that cannot ignore unknown values and
  preserving extensibility.

# Communication Failure Classifications

Communication failures caused by protocol ossification can take different
forms.  This document assigns a **communication manifestation** to each case.

Initial reachability failure:
: The first packet, request, or response does not pass, so that a connection or
  its first transaction cannot start.

Negotiation failure:
: A peer or path does not respond to, rejects, or incorrectly responds to a
  proposed version, capability, option, or extension.

In-flight dropping:
: Packets, including those after initial exchange, are dropped on the path.
  A case identifies whether this is one-way or two-way when known.

Field or option removal/rewrite:
: Communication might continue, but a middlebox removes, adds, or changes a
  header field, option, or payload.

Semantic or state failure:
: Packets arrive, but rewriting, incomplete interpretation, or endpoint state
  disagreement breaks a function or subsequent communication.

Silent fallback or feature degradation:
: Basic communication continues, but the new capability is not used and an
  older mechanism is used instead.

Failure of broad deployment:
: In addition to individual failures, the protocol or feature cannot safely be
  assumed available on the general Internet.

Intentional policy blocking:
: An operator explicitly does not permit a packet or feature under a security
  policy or similar policy.

# Cases: Issues, Reported Observations, and Mitigations

For each case, this document identifies the reported observation, its sources,
and the associated response.

## IP Layer

### Dropping IPv6 Extension Headers

**Issue:** Firewalls and other middleboxes drop packets containing IPv6
Extension Headers (EH), including standard Fragment Headers.  Variable
location of the Layer 4 header, arbitrary EH chains, and fragment
reassembly are common causes.

**Communication manifestation:** **Initial reachability failure**, **in-flight
dropping** (one-way or two-way), and sometimes **intentional policy blocking**.

**Report category:** **Measurement or operational observation.**

**Reported observations and sources:** {{RFC7872}} measured dropping of EH-containing
packets on the Internet.  {{RFC7045}} records widely deployed firewalls that
did not recognize standardized EHs or handle Fragment Headers.  {{RFC9098}}
also summarizes operational problems and reports of intentional dropping.

## Transport Layer

### Reachability of New IP Transport Protocols

**Issue:** NATs and stateful firewalls often implement and permit flow state
only for TCP and UDP.  An endpoint cannot assume that a new IP transport
protocol, such as SCTP or DCCP, will traverse the general Internet.

**Communication manifestation:** **Initial reachability failure**, **failure of
broad deployment**, and sometimes **intentional policy blocking**.

**Report category:** **Measurement or operational observation.**

**Reported observations and sources:** Edeline et al. {{EDELINE-UDP}} report RIPE Atlas and comparison
traffic measurements showing that middleboxes impede new transports and
extensions to existing transports.  WebRTC data channels specify SCTP over
DTLS over UDP rather than native SCTP {{RFC8841}}.

**Mitigation:** **Specified measure:** WebRTC data channels specify SCTP over
DTLS over UDP rather than native SCTP {{RFC8841}}.

### TCP Fast Open and TCP Option Intolerance

**Issue:** TCP Fast Open (TFO) cookies and data in SYN packets change
assumptions made by middlebox TCP state machines.  Operational deployments
observed post-handshake black holes and one-way data drops.

**Communication manifestation:** **Field or option removal/rewrite**,
**negotiation failure**, and **silent fallback or feature degradation**.

**Report category:** **Measurement or operational observation.** The
case was observed in an Apple service deployment on iOS 9 and OS X 10.11.

**Reported observations and sources:** Paasch {{PAASCH-TFO}}
shows a 30-second post-handshake black hole caused by middleboxes at some ISPs,
and one-way data dropping.  {{RFC9065}} also notes middleboxes that remove
unknown TCP options, and {{RFC7413}} specifies fallback behavior for TFO.

**Mitigation:** **Specified measure:** Treat TFO as
opportunistic and promptly retry with an ordinary TCP handshake.  **Observed
deployment action:** The Apple deployment used aggressive client timeouts to
blacklist a network for TFO and whitelisted a network after a successful TFO
connection.  It used TCP keepalive and receive sequence state to detect
one-way loss.

### TCP Sequence Translation and SACK Inconsistency

**Issue:** A middlebox that inserts or removes TCP payload can translate the
fixed sequence and acknowledgment fields but fail to translate SACK ranges.
Endpoints then receive invalid SACK information and loss recovery can degrade.

**Communication manifestation:** **Field or option removal/rewrite** and
**semantic or state failure**.

**Report category:** **Documented implementation behavior.**

**Reported observations and sources:** {{RFC9065}} identifies middleboxes that rewrite only
the fixed TCP header and not SACK information as an example of TCP
ossification.

### MPTCP and ECN

**Issue:** Paths that do not preserve MPTCP options such as MP_CAPABLE,
MP_JOIN, and DSS cause capability negotiation to fail and fall back to regular
TCP.  ECN-capable SYNs and ECE/CWR handling have similar compatibility risks.

**Communication manifestation:** **Negotiation failure**, **field or option
removal/rewrite**, and **silent fallback or feature degradation**.

**Report category:** **Mixed.** MPTCP includes **documented
implementation and operational behavior** in {{RFC8041}}.  ECN-capable SYN
fallback in {{RFC3168}} is a **specification consideration or design
constraint**.

**Reported observations and sources:** {{RFC8684}} specifies MPTCP fallback for middlebox
compatibility and {{RFC8041}} records operational heuristics.  {{RFC3168}}
specifies fallback for middleboxes that drop ECN-capable SYNs.

**Mitigation:** **Specified measure:** For MPTCP, fall back to TCP without MPTCP options when a
middlebox prevents connection establishment.  For ECN, retransmit the SYN
without ECN negotiation if an ECN-capable SYN receives no response.

### UDP Options: Mismatch between UDP Length and IP Length

**Issue:** UDP Options use the surplus area after the user data indicated by
the UDP Length field and before the end of the IP payload.  The UDP Length and
IP payload length therefore intentionally differ.  Some implementations and
inspection devices classify this difference as anomalous or an attack.

**Communication manifestation:** **Initial reachability failure** or
**in-flight dropping**.  An IDS alert alone does not necessarily cause failure.

**Report category:** **Measurement or operational observation** and
**documented implementation behavior.**

**Reported observations and sources:** {{RFC9868}}, Section 18, records interoperability
tests on Linux, macOS, Windows Cygwin, and NATs that delivered only user data,
but also reports embedded devices that delivered the entire IP datagram to a
UDP application.  It records the default configuration of the Alcatel-Lucent
"Brick" IDS, which reported a UDP/IP length mismatch as an attack.

**Mitigation:** **Specified measure:** Initially use only SAFE
Options so that legacy receivers retain the meaning of UDP user data.

### Unsafe UDP Options and Endpoint Ossification

**Issue:** UNSAFE Options for compression, encryption, fragmentation, and
similar functions can break semantics when a legacy receiver ignores the
option and processes user data.  UDP provides no standard stateful negotiation,
so a sender cannot know in advance that an endpoint supports such an option.

**Communication manifestation:** **Negotiation failure**, **semantic or state
failure**, or **silent fallback or feature degradation**.

**Report category:** **Specification consideration or design
constraint.**

**Reported observations and sources:** {{RFC9868}} defines SAFE Options as those that do not
change user-data meaning if ignored.  It deliberately defines incompatible
behavior for UNSAFE Options, for example by making user data empty and placing
payload in a FRAG Option.  The RFC also explains that endpoint support cannot
be known in advance.

**Mitigation:** **Specified measure:** Make a new option SAFE
where possible.  For UNSAFE Options, use
capability exchange, fallback, and reordering/loss handling at a higher layer,
and do not permit in-transit modification.

## TLS

### Version Intolerance

**Issue:** An old implementation can reject a ClientHello that advertises an
unknown newer TLS version rather than selecting a supported lower version.

**Communication manifestation:** **Negotiation failure** and **initial
reachability failure**.

**Report category:** **Documented implementation behavior.**

**Reported observations and sources:** {{RFC8446}} records version intolerance.  TLS 1.3
moves version preference to the `supported_versions` extension and fixes
`legacy_version` to the TLS 1.2 value, `0x0303`.

**Mitigation:** **Specified measure:** Do not depend on putting a
new version directly in an old version field; use a compatible negotiation
extension.

### ClientHello Extension Intolerance and TLS 1.3 Compatibility Mode

**Issue:** Endpoints and middleboxes can reject unknown TLS extensions, cipher
suites, lengths, or handshake ordering.  Some middleboxes treated the TLS 1.3
wire image as anomalous relative to TLS 1.2.

**Communication manifestation:** **Negotiation failure** and **initial
reachability failure**; compatibility mode can instead produce **silent
fallback or feature degradation**.

**Report category:** **Measurement or operational observation.**

**Reported observations and sources:** {{RFC8446}}, Appendix D.4, relies on field
measurements and specifies compatibility mode, including a dummy
ChangeCipherSpec.  Benjamin {{BENJAMIN-TLS13}}
reports a Chrome 63 rollout of TLS 1.3 draft 22 to 95% of stable users.  A
Canon PIXMA MX492 failed because BSAFE's private `extended_random` extension
number 40 collided with TLS 1.3 `key_share`.  Cisco Firepower in "Decrypt -
Resign" mode improperly forwarded `supported_versions`, `key_share`, and the
client random, breaking TLS 1.3 servers.  {{RFC8701}} defines TLS GREASE.
QUIC's use of TLS does not use the CCS compatibility mode {{RFC9001}}.

**Mitigation:** **Specified measure:** Clients should continuously use GREASE.

### Large PQC ClientHello Messages and TLS Inspection Intolerance

**Issue:** Hybrid post-quantum key exchange enlarges `supported_groups` and
`key_share` in ClientHello.  The ClientHello can span multiple TCP segments.
TLS inspection middleboxes that cannot reassemble and inspect it correctly can
make the handshake fail.

**Communication manifestation:** **Negotiation failure** and user-visible
**initial reachability failure**; disabling PQC key exchange produces **silent
fallback or feature degradation**.

**Report category:** **Measurement or operational observation.** The
sources identify concrete failures and fixes.

**Reported observations and sources:** Chromium CECPQ2 {{CECPQ2}}
records that larger TLS messages caused failures or timeouts in non-conformant
middleware during a CECPQ2 rollout, including FortiGate and possibly Palo Alto
Networks devices.  CECPQ2 used X25519 and NTRU-HRSS and is not the same
algorithm as current ML-KEM deployment, but is an operational precursor.
Fortinet's technical note {{FORTINET-MLKEM}}
documents `ERR_SSL_PROTOCOL_ERROR` and a fatal `illegal_parameter` alert for
ML-KEM ClientHello with Flow-based TLS Deep Inspection, with IPS Engine updates
as the long-term resolution.  Palo Alto Networks' technical note {{PALOALTO-PQC}}
records SSL session failure when ClientHello arrives in multiple packets over
an asymmetric path, with disabling the accumulation proxy or client PQC as
workarounds.

**Mitigation:** **Implemented measure:** Fortinet identifies an IPS Engine
update for Flow-based TLS Deep Inspection as the long-term resolution
{{FORTINET-MLKEM}}.

### Encrypted Client Hello and Middleboxes Assuming Plaintext SNI

**Issue:** ECH places the real SNI and related information in a
`ClientHelloInner` and sends a `ClientHelloOuter` containing an
`encrypted_client_hello` extension.  A middlebox that rejects unknown TLS
extensions, or an inspection/termination proxy that assumes visible SNI, can
fail the handshake.  Deliberately disabling ECH to restore plaintext SNI can
produce the same result, but is **intentional policy blocking**, not
ossification.

**Communication manifestation:** **Negotiation failure**, **initial
reachability failure**, or **silent fallback or feature degradation**.  A
deliberate ECH block is **intentional policy blocking**.

**Report category:** **Specification consideration or design
constraint.** {{RFC9849}} describes an incompatible TLS-terminating proxy as
capable of retry or connection failure depending on the client's trust
configuration.

**Reported observations and sources:** {{RFC9849}}, Section 6.2, specifies GREASE
`encrypted_client_hello` for clients without an ECHConfig.  Section 10.10.4
explicitly presents broad GREASE ECH deployment as a mitigation for network
ossification.  Section 8.1.2 explains how a conforming proxy ignores unknown
parameters and connects to the public name, and how failure can result when the
proxy certificate is not authoritative for that name.

**Mitigation:** **Specified measure:** Clients should continue GREASE ECH.

## DNS

### No Response or FORMERR to EDNS(0) OPT Records

**Issue:** An authoritative server, recursive resolver, or middlebox can give
no response, return FORMERR, or remove an EDNS(0) OPT pseudo-RR.  A resolver
may be unable to distinguish packet loss from EDNS intolerance and falls back
to plain DNS.

**Communication manifestation:** **Negotiation failure**, **in-flight dropping**
(query or response), and **silent fallback or feature degradation**.

**Report category:** **Measurement or operational observation.**

**Reported observations and sources:** {{RFC8906}} records widespread non-response to EDNS
queries, fallback to plain DNS, and possible DNSSEC validation failure.
{{RFC9170}} treats DNS extension intolerance as a case requiring broad
fallback.

**Mitigation:** **Specified measure:** Resolvers need EDNS fallback.

draft-ietf-dnsop-grease-03 {{DNSOP-GREASE}} proposes low-rate GREASE of DNS
extension points to expose intolerance to unknown values early.

### Individual EDNS Option Intolerance and UDP Fragmentation

**Issue:** Implementations that return FORMERR for an unknown EDNS option do
not distinguish lack of that option from lack of EDNS as a whole.  Large DNS
UDP responses that require IP fragmentation can time out when fragments are
dropped, including for DNSSEC resolution.

**Communication manifestation:** **Negotiation failure**, **in-flight dropping**
(primarily one-way on the response path), and **silent fallback or feature
degradation**.

**Report category:** **Documented implementation and operational
behavior.**

**Reported observations and sources:** {{RFC8906}} describes incorrect handling of unknown
EDNS options.  {{RFC8027}} describes EDNS-size fallback for DNSSEC reachability,
and {{RFC10001}} discusses oversized EDNS UDP payloads and fragmentation
reachability.

## HTTP

### HTTP Fields and WAF Allowlists

**Issue:** HTTP header fields, methods, status codes, and cache directives are
extension points.  A WAF, proxy, or cache that permits only known field names
or values can block, remove, or misinterpret a new field or a permitted new
value.

**Communication manifestation:** **Field or option removal/rewrite**,
**semantic or state failure**, or one-way **in-flight dropping** of a request or
response.

**Report category:** **Specification consideration or design
constraint.**

**Reported observations and sources:** draft-nottingham-http-grease {{HTTP-GREASE}}
identifies HTTP methods, status codes, header/trailer fields, cache directives,
content codings, and range units as potentially ossified extension points and
proposes GREASE.

## QUIC and HTTP/3

### Blocking UDP/443

**Issue:** In enterprise networks, access networks, or firewalls that block
UDP/443, QUIC Initial packets cannot complete a round trip and an HTTP/3
connection cannot start.

**Communication manifestation:** **Initial reachability failure**, **failure of
broad deployment**, and sometimes **intentional policy blocking**.

**Report category:** **Measurement or operational observation.** The
available measurement is not limited to UDP/443.

**Reported observations and sources:** The UDP reachability measurements in Edeline et al. {{EDELINE-UDP}}
show that UDP is not universally traversable.

### Middlebox Ossification on Google QUIC Public Flags

Google QUIC (GQUIC) in this section is the pre-IETF QUIC deployed and studied
by Google from 2013 and in the 2017 paper cited below.  It differs from IETF
QUIC in wire format and cryptographic handshake.

**Issue:** In October 2016, Google changed one public flag bit in a GQUIC packet
header.  A firewall product used that flag to identify and explicitly block
GQUIC.  Before the change, all GQUIC packets were blocked and the client could
fall back to TCP.  After the change, the classifier let the initial packet pass
but blocked later packets, creating a black hole after connection start and
preventing TCP fallback.

**Communication manifestation:** **In-flight dropping** after the first packet,
**semantic or state failure**, and **initial reachability failure**
when fallback does not activate.

**Report category:** **Measurement or operational observation.** Google
observed the failure during global deployment, rolled back the change, and
contacted the vendor.

**Reported observations and sources:** Section 7.5 of Langley et al. {{LANGLEY-QUIC}}
records the one-bit change, pathological loss of later packets in a firewall,
fallback failure, rollback, and a vendor classifier update.  It also describes
GQUIC's encryption of most transport-header information to reduce middlebox
modification and ossification.

**Mitigation:** **Observed deployment action:** In this case, client rollback
and the vendor classifier update restored service.  **Implemented measure:**
GQUIC encrypted transport information that did not require middlebox
interpretation.

### IETF QUIC Versions and Visible Invariants

IETF QUIC in this section is the standardized protocol based on {{RFC9000}}.
It is different from the GQUIC described in the preceding section.  The risk
of middleboxes fixing on a visible wire image is common to both.

**Issue:** A QUIC-aware middlebox can depend on its own interpretation of the
version-1 first packet, connection ID, fixed bit, or other visible values and
thereby interfere with version 2 or a future version.

**Communication manifestation:** **Negotiation failure**, **initial
reachability failure**, or **failure of broad deployment**.

**Report category:** **Specification consideration or design
constraint.** QUIC v2 exercises version negotiation to resist ossification.
Chaos Protection is an endpoint testing practice.

**Reported observations and sources:** {{RFC9369}} specifies QUIC v2 to exercise version
negotiation and counter an ossification vector, including middlebox attention
to the first packet.  {{RFC9000}} limits version-independent properties.

**Mitigation:** **Specified measure:** QUIC v2 exercises version negotiation to
mitigate ossification around version-1 Initial packets {{RFC9369}}.
**Implemented measure:** Chrome's QUIC implementation uses
"Chaos Protection" that splits ClientHello across CRYPTO frames and varies PING,
PADDING, and frame order; the quiche implementation {{QUICHE-CHAOS}} records
this practice.

## NTP

### NTPv4 Extension Field Intolerance

**Issue:** NTPv4 can append Extension Fields after its fixed header.  Existing
implementations that discard a packet with an unknown Extension Field cannot
interoperate with a peer sending a new field, although a receiver that does not
use the field would otherwise ignore it and continue time synchronization.

**Communication manifestation:** **Negotiation failure** or one-way/two-way
**in-flight dropping** of requests and responses; retry without the field can
be **silent fallback or feature degradation**.

**Report category:** **Documented implementation behavior.**

**Reported observations and sources:** {{RFC7822}} says a host SHOULD ignore an unknown
Extension Field, subject to policy.  {{RFC7821}} explicitly states that its
UDP Checksum Complement Extension Field cannot interoperate with legacy
implementations that discard unknown Extension Fields.

### Why NTPv5 Negotiation Cannot Use Extension Fields

**Issue:** NTPv5 is not wire-compatible with NTPv4.  Some widely deployed NTPv4
servers interpret a higher-version request as NTPv4 and copy its version number
into the response, or do not respond to a request containing an unknown
Extension Field.  NTPv4 Extension Fields therefore cannot be used for NTPv5
capability negotiation.

**Communication manifestation:** **Negotiation failure**, one-way **in-flight
dropping** of requests, and **failure of broad deployment**.

**Report category:** **Documented implementation behavior.** The source
records existing implementation behavior as a design input.

**Reported observations and sources:** draft-ietf-ntp-ntpv5-09, Section 12 {{NTPV5-DRAFT}}
records these behaviors.

**Mitigation:** **Specified measure:** NTPv5 places its negotiation signal in an
existing reference-timestamp field.

# Security Considerations

This document introduces no new security considerations.  Security
considerations for each case are in the RFCs, Internet-Drafts, and other
sources cited in the text.

# IANA Considerations

This document has no IANA actions.

--- back

# Acknowledgments
{:unnumbered}

The author thanks the authors and editors of the IETF specifications,
operational documents, and measurement studies cited in this document.

# References
{:unnumbered}
