---

title: "A Transport Services Mapping for QUIC"
abbrev: "QUIC Mapping"
docname: draft-taps-quic-mapping-latest
category: std

ipr: trust200902
area: Transport
workgroup: TAPS Working Group
keyword: Internet-Draft

stand_alone: yes
smart_quotes: no
pi: [toc, sortrefs, symrefs]

author:
 -
    name: Tommy Pauly
    organization: Apple
    email: tpauly@apple.com


--- abstract

This document defines a Transport Services API mapping for QUIC streams.

--- middle

# Introduction

This document defines a Transport Services mapping, as defined in {{!I-D.ietf-taps-impl}} for
the QUIC protocol {{!RFC9000}}. This mapping, presented in {{mapping}}, allows QUIC to be used
with the calls defined in the Transport Services API {{!I-D.ietf-taps-interface}}.

This mapping treats a single QUIC stream as a Transport Services Connection object, since this is an
equivalent abstraction to the byte-stream abstractions offered by TCP or TLS over TCP. QUIC streams are
multiplexed within QUIC connections; a QUIC connection is represented in the Transport Services API as
a Connection Group.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# QUIC Stream Mapping {#mapping}

Connectedness: Multiplexing Connected

Data Unit: Byte-stream

Connection Object: 
: A Connection object in the Transport Services API maps to a single QUIC stream between two hosts. This stream can be bidirectional or unidirectional.

Initiate: 
: Calling `Initiate` on a QUIC stream Connection causes the implementation to prepare a new QUIC stream to the Remote Endpoint. If there is already a QUIC connection to the Remote Endpoint, `Initiate` simply prepares a new stream by allocating a stream ID. If there is not already a QUIC connection established, the implementation will establish a connection first.

InitiateWithSend: 
: Early data sent in `InitiateWithSend` will be used for 0-RTT QUIC connection establishment, if the QUIC connection to the Remote Endpoint is not already established and the local device has previously negotiated support for 0-RTT establishment with the Remote Endpoint.

Ready: 
: A QUIC stream Connection is ready once the underlying QUIC connection is established, and once a stream ID can be allocated. This may be delayed if stream creation is blocked due to reaching the maximum streams limit.

InitiateError: 
: QUIC can throw various errors during connection setup (handshake failure, timeouts, etc). Errors for Initiate will represent QUIC connection-level errors. Once a QUIC connection is established, allocation of a QUIC stream ID may be delayed, but will not generate an error.

ConnectionError: 
: Once created, a QUIC stream Connection throws an error whenever the stream is disconnected, such as when a RESET_STREAM frame is receieved.

Listen: 
: Calling `Listen` for QUIC binds to a local UDP port and prepare to receive inbound QUIC connections and streams.

ConnectionReceived: 
: QUIC listeners will deliver each inbound QUIC stream as a Connection object. The relationship of inbound streams to other streams in a single QUIC connection can be detected by checking `Connection.GroupedConnections()`.

Clone: 
: Cloning a QUIC stream Connection creates a new stream on an existing QUIC connection. This new stream will inherently share all parameters with the original stream.

Send: 
: Sending data will generate a STREAM frame using the stream ID assigned to the Connection object.

Receive: 
: Calling `Receive` will indicate that the caller is ready to receive data from this stream, which is sent by the peer in STREAM frames using the stream ID assigned to the Connection object. Data is delivered in either the `Recieved` or `RecievedPartial` event.

Close: 
: Calling `Close` on a QUIC stream Connection indicates that the stream should gracefully closed by setting the FIN bit on the stream. 

Abort: 
: Calling `Close` on a QUIC stream Connection indicates that the stream should closed immediately, by sending a RESET_STREAM frame. 

CloseGroup: 
: Calling `CloseGroup` on any QUIC stream in a Connection Group indicates that the shared QUIC connection should be closed using a CONNECTION_CLOSE frame once all open streams have completed.

AbortGroup: 
: Calling `AbortGroup` on any QUIC stream in a Connection Group indicates that the shared QUIC connection should be closed immediately using a CONNECTION_CLOSE frame.

# Security Considerations

The security properties of a QUIC connection are expressed in the QUIC handshake, and thus are shared
amongst all streams on a single QUIC connection. When used with the Transport Services API, security
parameters are expressed in the Preconnection object. Connection objects used for QUIC streams MUST
only be grouped with other QUIC streams when the security parameters defined in the Preconnection
objects are identical or equivalent.

# IANA Considerations

This document has no IANA actions.
