---
sidebar_position: 1
sidebar_label: Overview
description: Short overview over all telnet options
"link": {
    "type": "generated-index",
    "description": "Overview of Telnet options relevant to MUDs"
  }
---
# Overview

*Telnet* is a protocol used to connect to remote computers in a terminal session. It started its live in the early 70's and became documented in [RFC 854](https://www.rfc-editor.org/rfc/rfc854) in 1983.

Telnet has a mechanism that enables client and server to exchange information *out-of-band*, meaning in a way that the content of this exchange is not part of the terminal session.
What kind of information is exchanged and how this is done is specificed in extra documents, one per feature: the **[Telnet Options](https://www.rfc-editor.org/rfc/rfc855.html)**. 
Since implementing those specifications is optional, every telnet session starts with a handshake in which both sides indicate which telnet options they support. 

## The Handshake

## List of options

The following list is the officially registered list of Telnet options. Be aware that MUD protocols also use Telnet options, but don't have officially registered numeric identifiers.

| Options | Name                               | Reference                                                    |
| ------- | ---------------------------------- | ------------------------------------------------------------ |
| 0       | Binary Transmission                | [[RFC856](https://www.iana.org/go/rfc856)]                   |
| 1       | Echo                               | [[RFC857](https://www.iana.org/go/rfc857)]                   |
| 2       | Reconnection                       | [NIC 15391 of 1973]                                          |
| 3       | Suppress Go Ahead                  | [[RFC858](https://www.iana.org/go/rfc858)]                   |
| 4       | Approx Message Size Negotiation    | [NIC 15393 of 1973]                                          |
| 5       | Status                             | [[RFC859](https://www.iana.org/go/rfc859)]                   |
| 6       | Timing Mark                        | [[RFC860](https://www.iana.org/go/rfc860)]                   |
| 7       | Remote Controlled Trans and Echo   | [[RFC726](https://www.iana.org/go/rfc726)]                   |
| 8       | Output Line Width                  | [NIC 20196 of August 1978]                                   |
| 9       | Output Page Size                   | [NIC 20197 of August 1978]                                   |
| 10      | Output Carriage-Return Disposition | [[RFC652](https://www.iana.org/go/rfc652)]                   |
| 11      | Output Horizontal Tab Stops        | [[RFC653](https://www.iana.org/go/rfc653)]                   |
| 12      | Output Horizontal Tab Disposition  | [[RFC654](https://www.iana.org/go/rfc654)]                   |
| 13      | Output Formfeed Disposition        | [[RFC655](https://www.iana.org/go/rfc655)]                   |
| 14      | Output Vertical Tabstops           | [[RFC656](https://www.iana.org/go/rfc656)]                   |
| 15      | Output Vertical Tab Disposition    | [[RFC657](https://www.iana.org/go/rfc657)]                   |
| 16      | Output Linefeed Disposition        | [[RFC658](https://www.iana.org/go/rfc658)]                   |
| 17      | Extended ASCII                     | [[RFC698](https://www.iana.org/go/rfc698)]                   |
| 18      | Logout                             | [[RFC727](https://www.iana.org/go/rfc727)]                   |
| 19      | Byte Macro                         | [[RFC735](https://www.iana.org/go/rfc735)]                   |
| 20      | Data Entry Terminal                | [[RFC1043](https://www.iana.org/go/rfc1043)][[RFC732](https://www.iana.org/go/rfc732)] |
| 21      | SUPDUP                             | [[RFC736](https://www.iana.org/go/rfc736)][[RFC734](https://www.iana.org/go/rfc734)] |
| 22      | SUPDUP Output                      | [[RFC749](https://www.iana.org/go/rfc749)]                   |
| 23      | Send Location                      | [[RFC779](https://www.iana.org/go/rfc779)]                   |
| 24      | Terminal Type                      | [[RFC1091](https://www.iana.org/go/rfc1091)]                 |
| 25      | End of Record                      | [[RFC885](https://www.iana.org/go/rfc885)]                   |
| 26      | TACACS User Identification         | [[RFC927](https://www.iana.org/go/rfc927)]                   |
| 27      | Output Marking                     | [[RFC933](https://www.iana.org/go/rfc933)]                   |
| 28      | Terminal Location Number           | [[RFC946](https://www.iana.org/go/rfc946)]                   |
| 29      | Telnet 3270 Regime                 | [[RFC1041](https://www.iana.org/go/rfc1041)]                 |
| 30      | X.3 PAD                            | [[RFC1053](https://www.iana.org/go/rfc1053)]                 |
| 31      | Negotiate About Window Size        | [[RFC1073](https://www.iana.org/go/rfc1073)]                 |
| 32      | Terminal Speed                     | [[RFC1079](https://www.iana.org/go/rfc1079)]                 |
| 33      | Remote Flow Control                | [[RFC1372](https://www.iana.org/go/rfc1372)]                 |
| 34      | Linemode                           | [[RFC1184](https://www.iana.org/go/rfc1184)]                 |
| 35      | X Display Location                 | [[RFC1096](https://www.iana.org/go/rfc1096)]                 |
| 36      | Environment Option                 | [[RFC1408](https://www.iana.org/go/rfc1408)]                 |
| 37      | Authentication Option              | [[RFC2941](https://www.iana.org/go/rfc2941)]                 |
| 38      | Encryption Option                  | [[RFC2946](https://www.iana.org/go/rfc2946)]                 |
| 39      | New Environment Option             | [[RFC1572](https://www.iana.org/go/rfc1572)]                 |
| 40      | TN3270E                            | [[RFC2355](https://www.iana.org/go/rfc2355)]                 |
| 41      | XAUTH                              | [[Rob_Earhart](https://www.iana.org/assignments/telnet-options/telnet-options.xhtml#Rob_Earhart)] |
| 42      | CHARSET                            | [[RFC2066](https://www.iana.org/go/rfc2066)]                 |
| 43      | Telnet Remote Serial Port (RSP)    | [[Robert_Barnes](https://www.iana.org/assignments/telnet-options/telnet-options.xhtml#Robert_Barnes)] |
| 44      | Com Port Control Option            | [[RFC2217](https://www.iana.org/go/rfc2217)]                 |
| 45      | Telnet Suppress Local Echo         | [[Wirt_Atmar](https://www.iana.org/assignments/telnet-options/telnet-options.xhtml#Wirt_Atmar)] |
| 46      | Telnet Start TLS                   | [[Michael_Boe](https://www.iana.org/assignments/telnet-options/telnet-options.xhtml#Michael_Boe)] |
| 47      | KERMIT                             | [[RFC2840](https://www.iana.org/go/rfc2840)]                 |
| 48      | SEND-URL                           | [[David_Croft](https://www.iana.org/assignments/telnet-options/telnet-options.xhtml#David_Croft)] |
| 49      | FORWARD_X                          | [[Jeffrey_Altman](https://www.iana.org/assignments/telnet-options/telnet-options.xhtml#Jeffrey_Altman)] |
| 50-137  | Unassigned                         | [[IANA](https://www.iana.org/assignments/telnet-options/telnet-options.xhtml#IANA)] |
| 138     | TELOPT PRAGMA LOGON                | [[Steve_McGregory](https://www.iana.org/assignments/telnet-options/telnet-options.xhtml#Steve_McGregory)] |
| 139     | TELOPT SSPI LOGON                  | [[Steve_McGregory](https://www.iana.org/assignments/telnet-options/telnet-options.xhtml#Steve_McGregory)] |
| 140     | TELOPT PRAGMA HEARTBEAT            | [[Steve_McGregory](https://www.iana.org/assignments/telnet-options/telnet-options.xhtml#Steve_McGregory)] |
| 141-254 | Unassigned                         |                                                              |
| 255     | Extended-Options-List              | [[RFC861](https://www.iana.org/go/rfc861)]                   |

## Telnet Today

