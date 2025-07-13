---
sidebar_label: SGA (3)
description: Control whether a client should show typed input or not.
---
# Supress Go-Ahead (SGA)

**Option code:** 3

**See also**: [RFC 858](https://www.rfc-editor.org/rfc/rfc858.html)

Old telnet implementations assumed operating a half-duplex connection. As a 
consequence once a party finished transmitting, it sent a telnet GA (Go Ahead)
command, to inform the other party that it now can send data.

Nowadays half-duplex isn't used anymore and therefore the usage of Go Ahead
messages not necessary. With this telnet option, both parties could agree on 
**not** using GA messages. 

In the context of MUDs the *Go Ahead* (GA, byte value 249) is sometimes used 
to indicate the end of a prompt.

| Tokens         | Bytes     | Meaning                                           |
| -------------- | --------- | ------------------------------------------------- |
| IAC WILL SUPPRESS-GO-AHEAD  | 255 251 3 | The party expresses its wish to not send GA messages. |
| IAC WON'T SUPPRESS-GO-AHEAD | 255 252 3 | The party expresses its wish to start sending GA messages. |
| IAC DO SUPPRESS-GO-AHEAD    | 255 253 3 | The party expresses its wish that the other party stops sending GAs. |
| IAC DON'T SUPPRESS-GO-AHEAD | 255 254 3 | The party expresses its wish that the other party starts sending GAs.  |

## Interworking with ECHO

When the (remote) [ECHO](echo) is enabled **and** the SGA telnet option is enabled too,
some MUD clients interprete this setup as the request not even to do local 
line buffering and send every keystroke via telnet. This *character-a-time-mode*.
is some kind of best practice, but not supported by all clients

