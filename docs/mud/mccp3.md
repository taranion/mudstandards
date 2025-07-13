---
sidebar_label: MCCP3 (Option 87)
---
# MUD Client Compression Protocol (MCCP3)

**Source**: [The MUD Client Protocol (MCP)](https://tintin.mudhalla.net/protocols/mccp)

## The MCCP3 Protocol

MCCP3 is a compression protocol which allows a MUD client to compress output to the receiving server using the zlib compression library. In some usecases this can significantly reduce bandwidth, while in the typical usecase it can provide security through obscurity because passwords and messages are no longer sent in plain text.
MCCP3 is implemented as a Telnet option RFC854, RFC855. The server and client negotiate the use of MCCP3 as they would any other telnet option. Once agreement has been reached on the use of the option, option sub-negotiation is used to start compression.

### Server Commands
```
IAC WILL MCCP3    Indicates the server supports MCCP3.
IAC WONT MCCP3    Indicates the server wants to disable MCCP3.
```
### Client Commands
```
IAC DO   MCCP3    Indicates the client supports MCCP3.
IAC DONT MCCP3    Indicates the client does not support MCCP3.
                  If the server has enabled compression it should disable it.
```
### Handshake
When a client connects to a server the server should send ``IAC WILL MCCP3``. The client should respond with either IAC DO MCCP3 IAC SB MCCP3 IAC SE or ``IAC DONT MCCP3``. If the client sends ``IAC DO MCCP3 IAC SB MCCP3 IAC SE`` it should immediately afterwards start compressing data.
### MCCP3 definitions
MCCP2 87

### Compression Format
Immediately after the client sends ``IAC SB MCCP3 IAC SE`` the client should create a zlib stream RFC1950.
Once compression is established, all client side communication, including telnet negotiations, takes place within the compressed stream.
The client may terminate compression at any point by sending an orderly stream end (Z_FINISH). Following this, the connection continues as a normal telnet connection.
The server may request ending compression at any point by sending ``IAC WONT MCCP3``. For example right before initiating a copyover.

### Compression Errors
If a decompression error is reported by zlib on the server side, the server can stop decompressing and send ``IAC WONT MCCP3``. The client in turn should disable compression upon receiving IAC WONT MCCP3, the connection continues as a normal telnet connection.
Alternatively the server can close the connection when a stream error is detected.

### MCCP versions
In 1998 MCCP used TELOPT 85 and the protocol defined an invalid subnogation sequence (IAC SB 85 WILL SE) to start compression. Subsequently MCCP version 2 was created in 2000 using TELOPT 86 and a valid subnogation (IAC SB 86 IAC SE).
As of 2004 virtually every MCCP enabled MUD client supports version 2, making version 1 obsolete. As such this specification only deals with version 2, and it is strongly discouraged for MUD servers to implement version 1.
In 2019 MCCP version 3 was created. This version does not replace MCCP 2 but is a separate protocol that allows the client to send compressed data to the server.