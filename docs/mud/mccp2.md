---
sidebar_label: MCCP2 (Option 86)
---
# MUD Client Compression Protocol (MCCP2)

**Source**: [The MUD Client Protocol (MCP)](https://tintin.mudhalla.net/protocols/mccp)

## The MCCP2 Protocol

MCCP2 is a compression protocol which allows a MUD server to compress output to the receiving client using the zlib compression library. This typically reduces the amount of bandwidth used by the server by 75 to 90%.
MCCP2 is implemented as a Telnet option RFC854, RFC855. The server and client negotiate the use of MCCP2 as they would any other telnet option. Once agreement has been reached on the use of the option, option sub-negotiation is used to start compression.

### Server Commands
```
IAC WILL MCCP2    Indicates the server supports MCCP2.
```
### Client Commands
```
IAC DO   MCCP2    Indicates the client supports MCCP2.
IAC DONT MCCP2    Indicates the client does not support MCCP2.
                  If the server has enabled compression it should disable it.
```
### Handshake
When a client connects to a server the server should send IAC WILL MCCP2. The client should respond with either IAC DO MCCP2 or IAC DONT MCCP2. If the server receives IAC DO MCCP2 it should respond with: IAC SB MCCP2 IAC SE and immediately afterwards start compressing data.

### MCCP2 definitions
MCCP2 86

### Compression Format
Immediately after the server sends IAC SB MCCP2 IAC SE the server should create a zlib stream RFC1950.
Once compression is established, all server side communication, including telnet negotiations, takes place within the compressed stream.
The server may terminate compression at any point by sending an orderly stream end (Z_FINISH). Following this, the connection continues as a normal telnet connection.

### Compression Errors
If a decompression error is reported by zlib on the client side, the client can stop decompressing and send IAC DONT MCCP2. The server in turn should disable compression upon receiving IAC DONT MCCP2, the connection continues as a normal telnet connection.
Alternatively the client can close the connection when a stream error is detected.
