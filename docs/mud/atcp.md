---
sidebar_label: ATCP (Option 200)
description: ATCP is a predecessor of GMCP and is commonly used to provide supplemental information or hints to the client by the server (and for the server to query capabilities and information from the client). 
slug: /ATCP_Specification
---
# Achaea Telnet Client Protocol (ATCP)
TELNET is the protocol used to connect to MUDs, however it is often not expressive enough for some of the functionality that a server may want to provide. For this reason, a separate protocol was designed that will piggy-back on top of normal TELNET (negotiated on and off like any other option) and allow a channel of communication between client and server that the user doesn’t necessarily have to see. This is commonly used to provide supplemental information or hints to the client by the server (and for the server to query capabilities and information from the client).

For instance, if a custom client was written to display vital character information in a separate window (or subwindow), then the server could send messages to the client encoding this character information. This could be done both invisible to the user and in a very efficient manner (to ease parsing, etc).

The content passed (as well as its format) is completely up to a system implementor. (With the one caveat that is explained below.)

#### Transport Protocol
This protocol needed to be compliant and invisible to existing TELNET standards because the primary client of choice is still simply a TELNET-compatible one (such as zMUD). So, the following format was designed:

## Enabling

The transport layer must be enabled. This involves using standard TELNET sub-option negotiation to determine if the server (and/or client) is capable of ATCP communication. RFC 855 http://www.faqs.org/rfcs/rfc855.html describes the process of sub-option negotiation. A summary is as follows. The server asks the client whether it is capable of supporting ATCP (a 3 byte sequence):
```
IAC DO ATCP (255 253 200) 
```
Note, that the byte 200 is an arbitrary value chosen by Iron Realms, it is not endorsed by any TELNET standards. It was chosen because it did not interfere with any other registered TELNET options. The client then responds with another 3 byte code:
```
IAC WILL ATCP (255 251 200) 
```
From this point on, the server knows that the client is ATCP capable (and is expecting to receive ATCP messages). This will be reflected in the node[].atcp property which will now be set to true. 

## Sending Messages
Once the ATCP option has been been enabled, a message may be sent to the 
client or server using standard TELNET sub-option markers. For example, 
``IAC SB ATCP <atcp message text> IAC SE``. The start and stop marker byte sequences are defined as follows.

| Char Sequence | Numeric Version | Description    |
| ------------- | --------------- | -------------- |
| IAC SB ATCP   | 255 250 200     | Start Sequence |
| IAC SE        | 255 240         | Stop Sequence  |

The text between the two markers includes the entire ATCP message body, and can be any text you desire (except, of course, the end marker sequence). You don’t have to worry about this when sending messages from server to client, however, because Rapture provides a very easy way to do it (see below).

## Message Format
The transport alone is not interesting. To provide useful functionality, we define a set of messages and an informal protocol that can be sent on this new piggy-backing transport.

We define a message to be a single block of text embedded in an ATCP begin/end sequence. The entire block of text is used as the message, with the following characteristics:

* Messages going to the server must be less than 2048 bytes in length.
* Lines are terminated with a single newline character ('\n')
* The first line is the "header" of the message.
  * The first word (contiguous, non-whitespace) is the message type.
  * Extra text may also be placed after the message type, and as long as it does not contain a newline, it is still considered part of the header. 
* All following lines (if any) are the message "content" 

To support flexibility and evolution of the standard, messages are grouped into "modules" which define what messages can be sent, what they mean, and what (if any) responses are defined for those messages. Each module is like a little "mini" protoco 

## Message Module Negotiation
To facilitate the flexible messaging protocol, we define a single message that is "outside" the module system and is provided to allow the client to tell the server about what modules it supports. This should be the very first message the client sends. (Note: currently there is no way to tell the -client- what features the server does, but this could easily be added by simply filtering the list and providing the last step in the negotiation.)
```
hello <client "key"> <client version> \n <supported module list>
```
Tell the server about the client including a "key", its version, and what modules it supports. The key is simply a single-word (congituous non-whitespace) describing the client, for instance "Nexus" or "MySuperCoolClient". The version is informational only, but needs to also be a single word (e.g. "1.2.3").

The module list is simply each module that the client supports, along with a version for that module (since modules themselves may change over time), one per line. The version of a module should be a simple integer value, starting with 1. So, an example message might be:
```
hello Nexus 3.0.0
composer 1
keepalive 1
filestore 2
buddylist 5
```

## Modules

See https://web.archive.org/web/20100330064816/http://www.ironrealms.com/nexus/atcp.html

From here on work in progress

### auth 1
Auth.Request CH

- [ATCP auth](https://web.archive.org/web/20100330083106/http://www.mudstandards.org/ATCP_auth)
- [ATCP composer](https://web.archive.org/web/20100330083106/http://www.mudstandards.org/ATCP_composer)
- [ATCP keepalive](https://web.archive.org/web/20100330083106/http://www.mudstandards.org/ATCP_keepalive)
- [ATCP char_name](https://web.archive.org/web/20100330083106/http://www.mudstandards.org/ATCP_char_name)
- [ATCP char_vitals](https://web.archive.org/web/20100330083106/http://www.mudstandards.org/ATCP_char_vitals)
- [ATCP room_brief](https://web.archive.org/web/20100330083106/http://www.mudstandards.org/ATCP_room_brief)
- [ATCP room_exits](https://web.archive.org/web/20100330083106/http://www.mudstandards.org/ATCP_room_exits)
- [ATCP mediapak](https://web.archive.org/web/20100330083106/http://www.mudstandards.org/ATCP_mediapak)
- [ATCP wiz](https://web.archive.org/web/20100330083106/http://www.mudstandards.org/ATCP_wiz)
- [ATCP java](https://web.archive.org/web/20100330083106/http://www.mudstandards.org/ATCP_java)
- [ATCP ping](https://web.archive.org/web/20100330083106/http://www.mudstandards.org/ATCP_ping)
- [ATCP unclassified](https://web.archive.org/web/20100330083106/http://www.mudstandards.org/ATCP_unclassified)

------
**Source:** [Internet Archive](https://web.archive.org/web/20100328130811/http://www.mudstandards.org:80/ATCP_Specification) of original mudstandards.org

