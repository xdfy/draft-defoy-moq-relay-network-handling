---
title: "MOQ Relays for Support of High-Throughput Low-Latency Traffic in 5G"
abbrev: MOQ-EXT-NET-HANDLING
docname: draft-defoy-moq-relay-network-handling-02
date: {DATE}
stream: IETF
category: std
ipr: trust200902
area: Transport
workgroup: MOQ

stand_alone: yes
pi: [toc, sortrefs, symrefs, docmapping]

author:
  -
    ins: X. de Foy
    name: Xavier de Foy
    org: InterDigital
    email: xavier.defoy@interdigital.com
    country: Canada
  -
    ins: R. Krishna
    name: Renan Krishna
    org: InterDigital
    email: renan.krishna@interdigital.com
    country: Canada
  -
    ins: T. Jiang
    name: Tianji Jiang
    org: China Mobile
    email: tianjijiang@chinamobile.com
    country: China

normative:

informative:
    I-D.draft-ietf-moq-transport:
    I-D.draft-ietf-mops-ar-use-case:

    TS26.522:
      title: '5G Real-time Media Transport Protocol Configurations'
      author:
      - ins: '3GPP'
      date: '2024'
      rc: '3GPP'
      target: 'www.3gpp.org/dynareport/26522.htm'

    TS23.501:
      title: 'System architecture for the 5G System'
      author:
      - ins: '3GPP'
      date: '2024'
      rc: '3GPP'
      target: 'www.3gpp.org/dynareport/23501.htm'

    TR23.700-70:
      title: 'Study on architecture enhancement for Extended Reality and Media service (XRM); Phase 2'
      author:
      - ins: '3GPP'
      date: '2024'
      rc: '3GPP'
      target: 'www.3gpp.org/dynareport/23700-70.htm'


--- abstract

This document describes a mechanism to convey information about media frames. The information is used for specific handling in functions such as error recovery and congestion handling. These functions can be critical to improve energy efficiency and network capacity in some (especially wireless) networks. Due to end-to-end encryption, MOQ relays are expected to extract the metadata required by these functions. This document aims to enable a level of wireless network support for MOQ equivalent to what is possible for RTP.


--- middle

# Introduction

Wireless networks can be a challenging environment for applications with high-throughput and low latency requirements, such as video conferencing and Extended Reality (XR, presented for example in {{I-D.draft-ietf-mops-ar-use-case}}). Wireless networks implement techniques to improve network capacity and energy efficiency, as well as reduce the impact of packet losses on users' quality of experience ({{intro-1-1}}). An extension to the RTP protocol has been defined, which enables metadata associated with application data units to be identified at the ingress point of the wireless network ({{intro-1-2}}). To enable a similar operation with the MoQ protocol {{I-D.draft-ietf-moq-transport}}, this document describes how a MOQ relay can be used at the ingress point of the wireless network ({{intro-1-3}}).

The rest of this document is structured as follows:

- {{section-xr-metadata}} describes XR metadata for MoQ, based on what is currently defined by 3GPP for RTP.
- {{section-xr-handling}} describes the behavior of the MOQ relay and of MOQ endpoints.
- {{section-sec}} describes applicable security considerations.


## Techniques used by Wireless Networks for XR Traffic {#intro-1-1}

The network can handle groups of packets based on how critical they are to the user's experience. Some groups of data packets hold application data units that are handled (e.g., decoded) together by the application. 3GPP defines the term "PDU set" to identify these groups of data packets carrying the payload of an application data unit [TS23.501], which can correspond to the data packets of an application layer data unit. The handling of application data units by the application can depend on other application data units (e.g., in the case of decoding dependency).
The wireless network performs differentiated handling of groups of data packets. For example, it prioritizes some groups over others in case of congestion. In congestion situations, the network can also selectively drop data packets that depend on an already lost data packets. Furthermore, the network can limit the amount of time that the radio needs to stay awake to transmit and receive data. To achieve this this, the scheduler can use information on the size and periodicity of traffic, as well as delay budget and maximum tolerable jitter specific to the application.

## Identifying XR Metadata in RTP Media Flows {#intro-1-2}

To perform differentiated handling of PDU sets, a network node (typically a User Plane Function, UPF) at the ingress point of a wireless network identifies PDU set metadata from a downlink media flow. 3GPP has developed an RTP header extension for XR traffic for this purpose (see Appendix A for details). When receiving a downlink packet, the UPF reads the XR metadata fields from the RTP header extension and transmits them to the RAN node in the GTP header that encapsulates the packet. The RAN node uses the XR metadata information to implement the differentiated handling described in {{intro-1-1}}.

## Identifying XR Metadata in MOQ flows {#intro-1-3}

For XR media traffic transported over the MOQ protocol, the UPF cannot access XR metadata unless it is exposed to the UPF in some fashion. This document describes how the UPF can act as, or communicate with, a MOQ relay to obtain XR metadata associated with media data. To enable this behavior, it is also necessary for the media sender to identify application data units that correspond to different desired traffic handling (e.g., different layers within a media frame), and provide associated metadata. {{figure-relay}} describes a UPF with a MOQ relay functionality, identifying XR metadata and transmitting it to access nodes, for example, for a media stream sent by B towards A and C. For privacy and security, it is desirable that the MOQ relay, which can be operated by a network or service operator, does not have access to media data. For interoperability, it is also desirable for these mechanisms to not be codec-specific. 


~~~~~~~~~~
  +---+  +-----------+            +-----------+
  | A |<-|Access Node|------------|           |
  +---+  |           |<-metadata--|           |            +---+
         +-----------+            |    UPF    |<--data+md--| B |
                                  | MOQ Relay |            +---+
         +-----------+            |           |
  +---+  |           |<-metadata--|           |
  | C |<-|Access Node|------------|           |
  +---+  +-----------+            +-----------+
~~~~~~~~~~
{: #figure-relay title="Traffic Handling by Access Networks using a MOQ Relay."}


# XR Metadata for 5G {#section-xr-metadata}

## List of Currently Defined XR Metadata

XR metadata applies to IP packets from the 5G system standpoint. Some metadata is the same for a group of IP packets (the PDU set), while some metadata varies per IP packet within the group. Some metadata relates to a data burst, which may consist of one or more PDU sets sent together as one burst of data.

3GPP defines metadata used for XR traffic handling:

- In the 3GPP release 18 (RTP header extension for PDU set marking, {{TS26.522}})
   - The *PDU Set Importance* (PSI, 4 bits) is the importance of this PDU Set compared to other PDU Sets within the same Multimedia Session.
   - The *end PDU of PDU set* (E, 1 bit) indicates the last PDU of the PDU set.
   - The *end of data burst* (D, 1 bit) indicates the last PDU of a data burst.
   - The *PDU set sequence number* (PSSN, 10 bits) identifies the PDU set. The PSSN is incremented monotonically by 1 for each subsequent PDU set.
   - The *PDU Sequence Number within a PDU Set* (PSN, 6 bits) identifies a PDU with the PDU set, starting from 0 and incremented monotonically for every PDU in the PDU set in the order of transmission from the sender.
   - The *PDU set size* (PSSize, 24 bits) is an optional field which indicates the total size, in bytes, of all PDUs of the PDU Set (including IP header and IP payload).
   - The *number of PDUs in the PDU Set* (NPDS, 16 bits) is an optional field which indicates the number of PDUs in the same PDU set.
- In the 3GPP release 19 (work in progress, in normative stage based on the conclusion of {{TR23.700-70}})
   - The *Burst Size* (BSize) describes the size of a burst of traffic, which includes one or more PDU sets.
   - The *Time to Next Burst* (TTNB) describes the time between the end of the present burst and the beginning of the next burst.

The optional RTP header extension fields are included subjects to an SDP signaling offer/answer negotiation.


## Applying XR Metadata to MOQ

Question #1: Should XR metadata for MOQ be worked on at the IETF or in 3GPP SA4 (like the RTP header extension work)? For now, the authors are considering the following approach:

- Define XR metadata for MOQ within the IETF MOQ WG, since the MOQ protocol is still in development and some questions (like extension mechanisms) can be best addressed by the protocol designers.
- To limit the need for 3GPP-specific expertise, restrict the initial definition of XR metadata for MOQ to what has been defined by 3GPP for RTP, in Release 18 {{TS26.522}}, and in Release 19 (assuming the Release 19 work will be completed by the time this draft is finalized).

Question #2: Should XR metadata be inferred from existing MOQ metadata or should XR metadata be explicit? For now, the authors are considering the following approach:

- Derive XR metadata if there is no practical way to signal it explicitly in MOQ (e.g., E and PSN in stream mode).
- Use explicit metadata in other cases, to associate metadata with a stream or datagram.
- Based on discussions in the MOQ WG, if a XR metadata field is determined to be exactly equivalent to an existing MOQ metadata field, in all use cases, then the equivalent existing MOQ metadata field will be preferred.

Our rationale for proposing this approach to question #2 includes:

- Metadata generation tools for the RTP header extension for XR can be reused to generate XR metadata for MOQ.
- It keeps XR metadata generation independent from the management of MOQ signaling, which reduces the risks for side effects and the need for heuristics in the MOQ relay to avoid those side effects. Future design evolutions of MOQ would also need to take into account its usage in 5G for XR.
   - For example, in datagram mode, the boundaries of a PDU set may be an object (which would be the default heuristic choice) but it could also be a group. Reusing MOQ object ID as PSSN for example, may not work in all cases.
   - In another example, the priority in MOQ may or may not always align with PDU set importance, because the priority is for the application layer while the importance is for differentiated QoS handling in the network. 
- Explicitly packaging XR metadata in MOQ can also simplify processing, since the suitability of a MOQ session for XR handling can be easily determined, based on the presence of explicit XR metadata.
- Some XR metadata is not covered by MOQ metadata today (e.g., BSize, TTNB) and needs to be explicit.
- Using explicit signalling, the sender can determine which part of the data flow is in a PDU set, and which part is not in a PDU set. 
  - For example, in a QUIC session that includes MOQ XR streams and other streams, control or other application traffic, the sender can explicitly mark the MOQ XR streams with XR metadata.
- Finally, XR metadata can be extended or modified from one 3GPP release to the next, and explicit versions of XR metadata for MOQ can be defined in sync with the RTP header extension for XR, independently from other MOQ protocol aspects.

### MOQ Datagram XR Metadata

When using MOQ in datagram mode, XR metadata is sent per datagram, since datagrams can be lost, and therefore sending per datagram is a way to ensure that the relevant metadata is always available when processing a datagram. 

The datagram XR header includes all XR metadata defined by 3GPP as part of the RTP header extension. 

- Mandatory fields:
  - E (1)
  - D (1)
  - PSI (4)
  - PSSN (10)
  - PSN (6)
- Optional fields are dependent on application-layer signalling to be present (e.g., based on indications in publication and subscription messages, while the support of these optional fields would be indicated in the catalog).
  - PSSize (24)
  - NPDS (16)
- Additional fields are conditional to completion of the RTP header extension work in Release 19 of 3GPP.
  - BSize should be present in the first PDU of a burst
  - TTNB should be signalled in the last PDU of a burst

### MOQ Stream XR Metadata

When using MOQ in stream mode, explicit XR metadata applies to the whole stream. There is little gain in providing a differentiated QoS for different portions of a same stream, since stream data is usable only when received in sequential order. Therefore, in stream mode, each stream should be a PDU set, which means XR metadata applies to the whole stream.

The stream's metadata includes explicit XR metadata defined by 3GPP as part of the RTP header extension, except for the fields which pertain to individual PDUs, which can be derived unambiguously from QUIC signalling (derivation of E and PSN is described in section 3.1).

- Mandatory fields:
   - PSI (4)
   - D (1)
   - PSSN (10)
- Optional fields are dependent on application-layer signalling to be present (e.g., new fields in publication and subscription messages, in catalog).
   - PSSize (24)
   - NPDS (16)
- Additional fields are conditional to completion of the RTP header extension work in Release 19 of 3GPP.
   - BSize should be present in the first stream of a burst
   - TTNB should be signalled in the last stream of a burst (e.g., if possible, in a trailing position in the stream).



# Traffic Handling of High-Throughput Low-Latency Traffic {#section-xr-handling}

## MOQ Relay Behavior {#moq-relay-behavior}

A MOQ relay at the ingress point of a wireless network extracts metadata associated with PDU sets.

In datagram mode, the relay obtains all XR metadata from the datagram XR header.

In stream mode, the relay obtains the XR metadata fields from the stream header. The MOQ relay derives the following XR metadata, which is not present in the stream header:

- E is derived from the FIN flag of the QUIC stream
- PSN is generated based on the QUIC/UDP/IP packets received in order of STREAM frame offset. In case there are missing packets, the relay can use the STREAM frame offset and path MTU to determine the sequence number of a packet received after one or more missing packets.

E and PSN are derived for each PDU of the stream.



## Endpoint Behavior

To enable XR traffic handling, a MOQ client should set up a MOQ connection through a MOQ relay providing this functionality. Discovery of such relay is out of scope of this document.

The MOQ server is configured to send XR metadata with a media stream. Both MOQ server and client can include an "xr-handling" field in (respectively) publication and subscription messages, to indicate support for sending and receiving XR metadata. The catalog can also include an "xr-handling" field to indicate support for XR metadata for a track. [TBD: use a mechanism to agree on specific optional metadata to be sent - including whether optional fields are included, and whether release 19 fields are included]

Prior to sending a datagram, an endpoint determines the XR metadata applicable to a datagram, and, if applicable, add a datagram XR metadata header in the datagram. [TBD: the datagram XR metadata header should be placed after the MOQ header? Need to define a mechanism to extend the MOQ datagram header].

Prior to sending a stream, an endpoint determines the XR metadata applicable to a stream, and, if applicable, add a stream XR metadata header in the stream. [TBD: the stream XR metadata header should be placed after the MOQ stream header? Need to define a mechanism to extend the MOQ stream header].



# Security Considerations {#section-sec}

To enable support for the feature described in this document, the application exposes metadata to a MOQ relay under the control of a network or service operator. End-to-end encrypted media is not exposed to the MOQ relay, so this is not seen as a high-risk exposure.

# Acknowledgments

Thanks to Srinivas Gudumasu, who was a contributor to the first revision of this draft. Thanks to Jaya Rao, Ghyslain Pelletier, John Kaippallimalil, Sri Gundavelli and Hang Shi for discussions and comments about this draft.

--- back

# XR RTP Header Extension for XR Traffic Handling in 5G Networks {#xr-in-5g}

3GPP defined an RTP header extensions for PDU set marking, in {{TS26.522}}, which enables a media sender to indicate PDU set metadata in each RTP packet.

## Release 18 XR Metadata

The fields defined in the Release 18 of 3GPP are included in this appendix, for the reader's convenience (text copied from {{TS26.522}} V18.1.0 with minor adaptations and omissions of some notes for readability):

- **End PDU of the PDU Set [E]** (1 bit): This field is a flag that shall be set to 1 for the last PDU of the PDU Set and set to 0 for all other PDUs of the PDU Set.
- **End of Data Burst [D]** (1 bit): This field is a flag that shall be set to 1 for the last PDU of a Data Burst. It shall be set to 0 for all other PDUs. A Data Burst may consist of one or more PDU Sets.
- **PDU Set Importance [PSI]** (4 bits): The PDU Set Importance field indicates the importance of this PDU Set compared to other PDU Sets within the same Multimedia Session. This information may help RAN to discard PDUs, when needed. Lower values shall indicate a higher importance PDU Set, with the highest importance PDU Set indicated by 1 and the lowest importance PDU Set indicated by 15. When the RTP sender cannot define an importance, it shall set the value to 0.
- **PDU Set Sequence Number [PSSN]** (10 bits): The sequence number of the PDU Set to which the current PDU belongs, acting as a 10-bit numerical identifier for the PDU Set. The PSSN shall be incremented monotonically by 1 for each subsequent PDU Set.

NOTE: This value wraps around at 1023, however, using the 16-bit RTP packet sequence number and PSSN pair, a receiver may uniquely distinguish between any PDU Sets.

- **PDU Sequence Number within a PDU Set [PSN]** (6 bits): The sequence number of the current PDU within the PDU Set. The PSN shall be set to 0 for the first PDU in the PDU Set and incremented monotonically for every PDU in the PDU Set in the order of transmission from the sender. 

NOTE:	A receiver may use the RTP packet sequence number together with the PSN to distinguish between PDUs within a PDU Set that contains more than 64 PDUs.

- **PDU Set Size [PSSize]** (24 bits): The PDU Set Size indicates the total size of all PDUs of the PDU Set to which this PDU belongs. This field is optional and subject to an SDP signaling offer/answer negotiation, where the RTP sender shall indicate whether it will provide the size of the PDU Set for that RTP stream. If not enabled, the field shall not be present within the RTP HE. If enabled, but the RTP sender is not able to determine the PDU Set Size for a particular PDU Set, it shall set the value to 0 in all PDUs of that PDU Set. The PSSize shall indicate the size of a PDU Set including RTP/UDP/IP header encapsulation overhead of its corresponding PDUs. The PSSize shall be expressed in bytes. It is recommended to add the PDU Set Size field when the Number of PDUs in the PDU Set field is present.

NOTE:	This field may be optionally present given the signaling of the "pdu-set-size" extension attribute in the SDP offer/answer negotiation.

- **Number of PDUs in the PDU Set [NPDS]** (16 bits): The number of PDUs within the PDU Set indicates the total number of PDUs belonging to the same PDU Set. This field is optional and subject to an SDP signaling offer/answer negotiation, where the RTP sender may indicate whether it will provide the number of PDUs within the PDU Set for that RTP stream. If enabled, but the RTP sender is not able to determine the Number of PDUs in the PDU Set, it shall set the value to 0 in all PDUs of that PDU Set.  It is recommended to add the Number of PDUs in the PDU Set field when the PDU Set Size field is present. 

NOTE: This field may be optionally present given the signaling of the "num-pdus-in-pdu-set" extension attribute in the SDP offer/answer negotiation.

## Release 19 XR Metadata

Potential additional fields in the Release 19 of 3GPP include (text based on work in progress in {{TR23.700-70}}):

- **Burst size [BSize]**: this field describes the size of a burst of traffic, which includes one or more PDU sets.
- **Time-to-next-burst [TTNB]**: this field describes the time between the end of the present burst and the beginning of the next burst.


