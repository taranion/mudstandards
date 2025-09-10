---
sidebar_label: MCCP4 (Option 88)
---
# MUD Client Compression Protocol (MCCP4)
	Draft: draft-rahjiii-MCCPX-04.md
	Obsoletes: draft-rahjiii-MCCPX-03.txt
	Published: 9 September 2025
	Intended Status: Informational
	Expires: 9 March 2026
	Authors: Asmodeus, RahjIII

MUD Client Compression Protocol Extensions - MCCPX
==================================================

## Abstract

This document describes an implementation of the MCCPX compression negotiation
protocol for Telnet based MUD Servers and Clients.

## Discussion Venues

Discussion of this document takes place on the /r/MUD Discord channel
#MudStandards [DISCORD1].

## Status of This Memo

This document describes the rough consensus used in the creation of running
code for the protocol.  It is inappropriate to use this document as reference
material or to cite it other than as "work in progress."

## Table of Contents

1. Introduction
2. Definitions
    1. Terminology
    2. Message Codes
    3. MCCPX Messages
    4. MCCPX Compression Encodings
3. Protocol Overview
4. Protocol Phases
5. Implementation Details
6. Example Negotiations
7. Acknowledgements
8. Contributors
9. Index

## 1. Introduction

The MCCPX TELNET extension protocol allows for bi-directional compression and
decompression of all traffic sent between the Telnet server and client, using a
negotiated compression algorithm.

MCCPX is designed to reduce bandwidth usage while maintaining low latency for
interactive text-based gaming.  While its expected use is for interactive text
based games, the protocol is applicable for any Telnet use.

Like the MCCP2 [MCCP2] compression protocol, MCCPX is explicitly unidirectional.
Successfully negotiated compression is applied to data transmitted from the
sending side, and does NOT have any effect on data transmitted back by the
receiving side.  Unlike MCCP2, MCCPX is intended to be Telnet server/user
agnostic, and may be offered by either peer in the Telnet session wishing to
transmit compressed data. Most importantly, the MCCPX protocol provides
negotiation for different compression algorithms using IANA standard names, and
is intended to support future compression algorithms as they are developed.

This specification defines the complete MCCPX protocol, including negotiation
sequences, data formats, and implementation requirements.

## 2. Definitions

### 2.1. Terminology

**Telnet** [STD8] is widely used as the communication protocol for
interactive text games, commonly known as MUD, MU\*, MUSH, or other names.  In
this document, "**MUD**" will be used to mean all interactive terminal text
games that run over the Telnet protocol, with the understanding that not all of
them are "Multi-User Dungeons".

The terms Telnet "**Server**" and MUD "**Game**" will be used to indicate the
Telnet peer that listens for and answers connection requests.  The terms Telnet
"**User**" and MUD "**Client**" will be used to indicate the Telnet peer that
initiates connection requests.

A Telnet data "**Stream**" refers to the one way transfer of data sent from one
peer and received by the other.

In this protocol, a peer that offers to compress its data stream is called the
"**Compressor**".  The side that accepts a compressed data stream is called the
"**Decompressor**".

In Telnet terms, this means that the "server" may offer to compress the data it
sends to the "user".  In this case, the Telnet "server" is a Compressor, and
the Telnet "user" is a Decompressor.  If the "user" offers to compress the data
that it sends to the "server", the "user" also becomes a Compressor and the
"server" is also becomes a Decompressor.

In MUD terms, this means that the MUD "game" may offer to compress the data it
sends to the MUD "Client".  In this case, the MUD "game" is a Compressor, and
the Telnet "user" is a Decompressor.  If the "client" offers to compress the
data that it sends to the "game", the "client" also becomes a Compressor and
the "game" also becomes a Decompressor.

### 2.2. Message Codes

The message codes used by the MCCPX protocol are as follows:

| Standard Telnet Codes |     |
|-----------------------|-----|
| IAC                   | 255 |
| DONT                  | 254 |
| DO                    | 253 |
| WONT                  | 252 |
| WILL                  | 251 |
| SB                    | 250 |
| SE                    | 240 |

| TELOPT Code |    |
|-------------|----|
| `MCCPX`     | 88 |

| MCCPX Subnegotiation Codes |                    |
|----------------------------|--------------------|
| `MCCPX_ACCEPT_ENCODING`    | 1                  |
| `MCCPX_BEGIN_ENCODING`     | 2                  |
| `MCCPX_WONT`               | 252                |
| `UNKNOWN`                  | (all other values) |

### 2.3. MCCPX Messages

To maintain clear symmetry and to avoid ambiguity during bi-directional
protocol negotiation, the Compressor and Decompressor are limited to sending
specific messages.

The Compressor MAY ONLY send:

- `IAC WILL MCCPX`
- `IAC WONT MCCPX`
- `IAC SB MCCPX MCCPX_BEGIN_ENCODING <encoding> IAC SB`
- `IAC SB MCCPX MCCPX_WONT <subnegotiation code> IAC SB`

The Decompressor MAY ONLY send:

- `IAC DO MCCPX`
- `IAC DONT MCCPX`
- `IAC SB MCCPX MCCPX_ACCEPT_ENCODING <encoding list> IAC SB`
- `IAC SB MCCPX MCCPX_WONT <subnegotiation code> IAC SB`

### 2.4. MCCPX Compression Encodings

The currently defined compression encodings are:

| Encoding | Description                         |
|----------|-------------------------------------|
| deflate  | via the zlib compression library    |
| zstd     | via the zstd compression library    |
| none     | a 1:1 pass-through used for testing |

It is expected that names of future compression encodings should follow the
Http content encodings from IANA http-parameters content-coding [IANA1].

Implementers MAY choose whatever name they wish for compression algorithms, but
this protocol RECOMMENDS using the "x-name" style for experimental protocols
that are not defined here, or by IANA.

The list of supported compression encodings are COMMA ',' separated and are
listed in a most-preferred to least-preferred order. Note that whitespace after
the comma is NOT allowed.

## 3. Protocol Overview

MCCPX operates as a telnet option that allows a peer to compress outgoing data
after successful negotiation of a compression algorithm with the other peer.
The protocol consists of three phases:

1. **Negotiation Phase**: Compressor offers MCCPX, Decompressor accepts and
declares supported encodings

2. **Activation Phase**: Compressor selects encoding and begins compression

3. **Data Phase**: All subsequent data in the stream sent by the Compressor is
compressed using the selected algorithm.

4. **Shutdown Phase**: Upon request from the Decompressor, or as desired by the
Compressor, the compressed stream is terminated in an orderly fashion as
provided by the compression algorithm.

## 4. Protocol Phases

The following is a description of the MCCPX negotiation sequence for a
Compressor that wishes to offer a compressed data stream to its peer (the
Decompressor).  For the MCCPX protocol, it does not matter if the Compressor is
a "server", "user", "game", or "client", as the protocol flow is the same in
each case.  Note that as with other Telnet Option protocols, MCCPX may be
negotiated in parallel with other Telnet options, and simultaneously by a peer
that also wishes to act as a Compressor.

```
Compressor:		Decompressor:

[1. Negotiation Phase]
IAC WILL MCCPX
(I am willing to negotiate with you about MCCPX)

				IAC DO MCCPX 
				(Please do.)
				IAC SB MCCPX ACCEPT_ENCODING "list of encodings" IAC SE
				(I speak any of these, in order of preference)
				-- or --
				IAC DONT MCCPX 
				(No thank you.)

[2. Activation Phase]
IAC WONT MCCPX
(Sorry, we don't have a common encoding, i will not do MCCPX with you.)
-- or --
IAC SB MCCPX BEGIN_ENCODING "encoding" IAC SE
(I choose to use the this compression encoding from your list, and
following the IAC SE, everything I send will be compressed.)

[3. Data Phase]
(Compressed Telnet Data Stream)
				(Normal Telnet Data Stream)
```

At any point after compression has started, Compressor may initiate an orderly
shutdown:

```
Compressor:		Decompressor:

[4. Shutdown Phase]
(I want to stop compressing.)

(sent data is still compressed until the compression stream is ended in an
orderly fashion that the Decompressor is expected to understand and detect.)

(Immediately after ending compression, Compressor sends:)

IAC WONT MCCPX
(I have stopped doing MCCPX.)

(after shutdown, sender MAY offer to restart negotiation.)

IAC WILL MCCPX
(... but I am still willing to negotiate with you about MCCPX)

```

At any point after compression has started, the Decompressor may request that
the Compressor initiate an orderly shutdown:

```
Compressor:		Decompressor:

				IAC DONT MCCPX
				(I want to stop decompressing your stream, Please stop the
				compressed stream in an orderly fashion as soon as you get this
				message.)

(Compressed data is continued to be sent until the Compressor ends the stream
in an orderly fashion that the Decompressor is expected to understand and
detect, as shown above.)

```

During subnegotiation, either side may send an UNKNOWN subnegotiation code that
MUST be answered with a negative acknowledgement `MCCPX_WONT` message.

```
Compressor:		Decompressor:

IAC SB MCCPX UNKNOWN "makes sense to me" IAC SE
(I know what UNKNOWN means, and I want to do it.)

				IAC SB MCCPX MCCPX_WONT UNKNOWN IAC SB
				(Sorry, I don't understand the UNKNOWN)
```

Note that the value of the UNKNOWN code in the response MUST match the value of
the UNKNOWN code sent in the request.

## 5. Implementation Details

### Bi-Directional Compression

To obtain bi-directional compression, the unidirectional MCCPX protocol MUST be
negotiated for each sending side of a telnet connection independently.  A peer
wishing to become a Compressor makes that known with an unsolicited "WILL
MCCPX" offer.

It is expected, understood, and not in any way an error for a MUD Game to
negotiate compression for the data that it sends to the MUD Client, while
refusing to accept negotiation for compressed data back from the MUD Client, or
for the Mud Client to simply never offer to compress the data it sends to the
MUD Game.  While the MCCPX protocol MAY be implemented bi-directionally, it is
NOT REQUIRED to be implemented bi-directionally.

Some Telnet "user" or "MUD Client" implementations may choose to operate in a
"passive negotiation mode" for compatibility with non-telnet speaking servers.
In this mode, they do not immediately offer to negotiate any Telnet protocols
that the "server" or "game" has not offered first.  It is understood that some
telnet "user" or "client" implementations will not offer to act as a Compressor
until after the "server" or "game" has offered MCCPX negotiation itself.
Implementations operating in the standard (active, non-passive) negotiation
mode may offer to become an MCCPX Compressor at any time.

### Extensibility

The protocol is extensible, and new MCCPX subnegotiation codes that are UNKNOWN
today are expected to be defined in the future. An MCCPX implementation MUST
respond to an unrecognized suboption negotiation code with an appropriate MCCPX
suboption `MCCPX_WONT` message.  Because it is understood that future suboption
codes may or may not be designed with a specific acknowledgement, all UNKNOWN
codes MUST elicit a negative acknowledgement with the `MCCPX_WONT` message.  The
negative acknowledgement should not be considered a fatal error by the sender.
It is only an indication that the suboption was not recognized, and it is left
to the sender of the UNKNOWN suboption to decide how to handle the issue.

### Compression Algorithm Support

The MCCPX protocol does NOT require support for any specific compression
algorithm. An implementation MAY provide any, all, or none of the MCCXP defined
compression protocols, in any order it prefers.

Because prior MUD Telnet art already includes Telopt protocols for negotiation
the 'deflate' algorithm, the MCCPX protocol recommends that MUD Clients and
Servers offer preferential support for "zstd,deflate,none" compression
encodings.

Because the MCCPX protocol is negotiated independently by each side wishing to
act as a Compressor, it is understood that peers may choose to use different
compression encodings for each stream direction.  For example, a "game" may
negotiate a general compression algorithm such as 'deflate' for its sent data,
while a "MUD Client" may negotiate a different algorithm that is better suited
for compressing the sparse set of game commands that it expects to send.

#### Zstandard 'zstd' Compression Details

MCCPX zstd [RFC8878] compression recommends the following parameters:

- **Compression Level**: Typically 8-9 (configurable 1-22)
- **Streaming Mode**: `ZSTD_e_flush` for real-time data
- **Frame Format**: Standard Zstd frames with headers
- **Window Size**: Default (determined by compression level)
- **Frame Boundaries**: Each compression operation may produce partial frames
- **Flushing**: Data is flushed after each logical message to ensure low latency
- **Termination**: Streams are properly terminated with `ZSTD_e_end` when
  compression ends

#### ZLIB 'deflate' Compression Details

MCCPX 'deflate' [RFC1951] compression recommends the following parameters:

- **Streaming Mode**:  `Z_SYNC_FLUSH` for real-time data
- **Frame Boundaries**: Each compression operation may produce partial frames
- **Flushing**: Data is flushed after each logical message to ensure low latency
- **Termination**: Streams are properly terminated with `Z_FINISH` when compression ends

#### 'None' Compression Details

MCCPX 'none' compression is for debugging the MCCPX protocol implementations.
It is recommended that outside of debugging, it is preferable to agree not to
do MCCPX compression at all, rather than to agree to do MCCPX with the 'none'
encoding.

- **Streaming Mode:** data is sent 'as-is', and does not require any
  decompression.

### Compatibility With Other Compression Telopts

As with prior art Telnet compression protocols such as COMPRESS, MCCP2, and
MCCP3, the MCCPX protocol provides a compression wrapper around a single Telnet
transmission stream.  "Stacking" of compression algorithms before or after
MCCPX negotiation IS NOT SUPPORTED.

The MCCPX Telnet option MUST NOT be negotiated or applied to a stream whenever
any non-MCCPX Telopt compression stream wrapper is already being negotiated
for, or is applied to the stream.

Implementations wishing to provide MCCPX compression MUST offer to negotiate it
before offering to negotiate any other compression options, and MUST NOT agree
to negotiate other compression protocols while MCCPX is active.

An implementation MAY choose to allow negotiation of a non-MCCPX compression
option (such as MCCP2, etc) after MCCPX negotiation has failed, but is NOT
REQUIRED to do so.  Therefore, a "server" or "game" implementation that
supports both MCCPX and MCCP2 compression SHOULD offer to use MCCPX "deflate"
compression instead of relying on MCCPX negotiation failure to fall back onto
MCCP2.

### Buffer Management

Compressors and Decompressors should implement appropriate buffering, and be
aware that many compression algorithms can produce compressed output that is
larger than its pre-compressed input.  Compression algorithms tend to be more
efficient when given larger blocks to compress.  It is usually preferable to
send as many lines as possible per message before flushing the buffer.  For
example, it is better for a MUD game to send all of the lines of text that
describe a 'room' together in one buffer, rather then to send each line of text
individually.

- **Input Buffer**: Raw text to be compressed
- **Output Buffer**: Compressed data ready for transmission
- **Flush Policy**: Flush after each complete message or prompt
- **Buffer Sizes**: Recommended 16KB for output buffers, or as recommended by
  the compression algorithm.

### Telnet Command Handling

**Important**: Once compression is active, Telnet IAC sequences within the
compressed stream must be handled carefully:

- IAC bytes in compressed data do NOT need escaping
- The compression layer handles all byte values transparently
- Clients must decompress the stream before interpreting Telnet commands

### Security Concerns

MCCPX Compression provides no additional security functions for the Telnet
Protocol.  While it may be tempting to consider providing an MCCPX compression
encoding that implements an encryption layer, MCCPX implementations SHOULD NOT
do so, and are STRONGLY ADVISED to use the Telnet-SSL, also known as the
"telnets - protocol for telnet protocol over TLS/SSL" that is assigned to IANA
well-known tcp port 992. [IANA2] Note that "telnets" is NOT to be confused with
the Telnet START-TLS Option described in INTERNET-DRAFT
draft-altman-telnet-starttls-02.txt [ALTMAN] and later documents, as STARTTLS
is considered vulnerable to hijacking attacks and should be avoided.

For the Telnet-SSL protocol, MCCPX compression is applied after SSL encryption,
and before the Telnet protocol, as it would be in the unencrypted Telnet case.

Compression encoding names should be validated before use.  There is no
predefined length for the encoding list or for a single encoding name.  The
format for encoding names follow "RFC 9110 HTTP Semantics" [STD97].  See
[STD97] Sections 2,5 and 17 for information on field lengths and corresponding
security concerns.

## 6. Example Negotiations

### Example 1:

Game server side supports zlib (deflate) compression on send, but does not
support client side compression.  Client is MCCPX aware, supports only server
side zlib compression, and won't compress player's commands. (equivalent to
most typically encountered server side MCCP2 setup)

```
Flow #1
Game:			Client:
IAC WILL MCCPX
				IAC DO MCCPX
				IAC SB MCCPX ACCEPT_ENCODING "zstd,deflate" IAC SE
IAC SB MCCPX BEGIN_ENCODING "deflate" IAC SE
(sent data is now compressed)
```

### Example 2:

Game server side supports zlib compression on send, but does NOT support client
side compression.  Client is MCCPX aware, supports zlib and zstd compression,
and would be willing to compress player commands for the server.

```
Flow #1
Game:			Client:
IAC WILL MCCPX
				IAC DO MCCPX
				IAC SB MCCPX ACCEPT_ENCODING "zstd,deflate" IAC SE
IAC SB MCCPX BEGIN_ENCODING "deflate" IAC SE
(sent data is now compressed)

Flow #2
Client:			Game:
IAC WILL MCCPX
				IAC DONT MCCPX

Flows shown with one possible interleaving:
Game:			Client:
IAC WILL MCCPX
				IAC WILL MCCPX
				IAC DO MCCPX
				IAC SB MCCPX ACCEPT_ENCODING "zstd,deflate" IAC SE
IAC DONT MCCPX
IAC SB MCCPX BEGIN_ENCODING "deflate" IAC SE
(sent data is now compressed)
```

### Example 3:

Both sides support zstd compression, bi-directional compression is desired.

```
Flow #1
Game:			Client:
IAC WILL MCCPX
				IAC DO MCCPX
				IAC SB MCCPX ACCEPT_ENCODING "zstd,deflate" IAC SE
IAC SB MCCPX BEGIN_ENCODING "zstd" IAC SE
(sent data is now compressed)

Flow #2
Client:			Game:
IAC WILL MCCPX
				IAC DO MCCPX
				IAC SB MCCPX ACCEPT_ENCODING "zstd,deflate" IAC SE
IAC SB MCCPX BEGIN_ENCODING "zstd" IAC SE
(sent data is now compressed)

Flows shown with one possible interleaving:
Game:			Client:
IAC WILL MCCPX
				IAC WILL MCCPX
				IAC DO MCCPX
				IAC SB MCCPX ACCEPT_ENCODING "zstd,deflate" IAC SE
IAC DO MCCPX
IAC SB MCCPX ACCEPT_ENCODING "zstd, deflate" IAC SE
IAC SB MCCPX BEGIN_ENCODING "zstd" IAC SE
(sent data is now compressed)
				IAC SB MCCPX BEGIN_ENCODING "zstd" IAC SE
				(sent data is now compressed)
```

### Example 4:

Game only supports new experimental "cruncher" protocol, client supports
zstd, deflate, and experimental "masher" protocol.  There are no compression
encodings in common.

```
Flow #1
Game:			Client:
IAC WILL MCCPX
				IAC DO MCCPX
				IAC SB MCCPX ACCEPT_ENCODING "zstd,x-masher,deflate" IAC SE
IAC WONT MCCPX
(nothing I can do is acceptable, so no compression)

Flow #2
Client:			Game:
IAC WILL MCCPX
				IAC DO MCCPX
				IAC SB MCCPX ACCEPT_ENCODING "x-cruncher" IAC SE
IAC WONT MCCPX
(nothing I can do is acceptable, so no compression)

Possible combined:
Game:			Client:
IAC WILL MCCPX
				IAC WILL MCCPX
				IAC DO MCCPX
				IAC SB MCCPX ACCEPT_ENCODING "zstd, x-masher, deflate" IAC SE
IAC DO MCCPX
IAC SB MCCPX ACCEPT_ENCODING "x-cruncher" IAC SE
IAC WONT MCCPX
(nothing I can do is acceptable, so no compression)
				IAC WONT MCCPX
				(nothing I can do is acceptable, so no compression)
```

## 7. Acknowledgements

## 8. Contributors

Asmodeus, Mfontani, Morwen, RahjIII, Vadi

## Index
- [ALTMAN] - <https://datatracker.ietf.org/doc/html/draft-altman-telnet-starttls-02>
- [DISCORD1] - <https://discord.com/channels/279748146316312576/1393998251786768384>
- [IANA1] - <https://www.iana.org/assignments/http-parameters/http-parameters.xhtml#content-coding>
- [IANA2] - <https://www.iana.org/assignments/service-names-port-numbers/service-names-port-numbers.xhtml>
- [MCCP2] - <https://tintin.mudhalla.net/protocols/mccp/>
- [RFC1951] - <https://datatracker.ietf.org/doc/html/rfc1951>
- [RFC8878] - <https://datatracker.ietf.org/doc/html/rfc8878>
- [STD8] - <https://datatracker.ietf.org/doc/std8/>
- [STD97] - <https://datatracker.ietf.org/doc/html/rfc9110>

## Author's Address
