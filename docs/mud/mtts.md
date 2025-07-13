---
sidebar_label: MTTS (Option 24)
---
# MUD Terminal Type Standard

MUD servers often want to know the various capabilities of a Mud Client. While  various methods exist there is no consistent nor guaranteed way to do  so.

The Mud Terminal Type Standard seeks to address  these issues by providing a transparant and straight forward standard  for Mud Clients to communicate their terminal capabilities. It does so  by extending and standardizing RFC1091 which describes the Telnet  Terminal-Type Option.

## The TTYPE Protocol

TTYPE is implemented as a Telnet option RFC854, RFC855. The server and client negotiate the use of the TTYPE option as they would any other telnet option as detailed in RFC1091. Once agreement has been reached on the use of the option, option sub-negotiation is used to exchange information between the server and client.

### Server Commands
```
IAC DO   TTYPE    Indicates the server wants to receive TTYPE information.
IAC DONT TTYPE    Indicates the server wants to discontinue receiving TTYPE information.
```

### Client Commands
```
IAC WILL TTYPE    Indicates the client will send TTYPE information.
IAC WONT TTYPE    Indicates the client will not send TTYPE information.
```

### Handshake
When a client connects to a TTYPE enabled server the server should send IAC DO TTYPE. The client should respond with either IAC WILL TTYPE or IAC WONT TTYPE. Once the server receives IAC WILL TTYPE the client and the server can send TTYPE sub-negotiations.
The client should never initiate a negotiation, in the case this does happen the server should abide by the state change. To avoid trigger loops the server should not respond to negotiations from the client, unless it correctly implements the Q method in RFC 1143.

### Disabling TTYPE
When a typical MUD server performs a copyover it loses all previously exchanged TTYPE data. If this is the case, before the actual copyover, the MUD server should send IAC DONT TTYPE, the client in turn should fully disable and reset its TTYPE state. After the copyover has finished the server and client behave as if the client has just connected, so the server should send IAC DO TTYPE.

## TTYPE definitions
```
TTYPE            24

IS                0
SEND              1
```

## Example TTYPE handshake
```
server - IAC DO TTYPE
client - IAC WILL TTYPE
server - IAC   SB TTYPE SEND IAC SE
client - IAC   SB TTYPE IS   "TINTIN++" IAC SE
server - IAC   SB TTYPE SEND IAC SE
client - IAC   SB TTYPE IS   "XTERM" IAC SE
server - IAC   SB TTYPE SEND IAC SE
client - IAC   SB TTYPE IS   "MTTS 137" IAC SE
server - IAC   SB TTYPE SEND IAC SE
client - IAC   SB TTYPE IS   "MTTS 137" IAC SE
```
The quote characters mean that the encased word is a string, the quotes themselves should not be send.

## Mud Client Name
As RFC1091 details the server can request TTYPE information using sub-negotiations. On the first TTYPE SEND request the client should return its name, preferably in all caps. Appending the client version is optional.

## Mud Client Terminal Type
On the second TTYPE SEND request the client should return a terminal type, preferably in all caps. Console clients should report the name of the terminal emulator, other clients should report one of the four most generic terminal types.

| Value | Description                                                  |
| ----- | ------------------------------------------------------------ |
| DUMB  | Terminal has no ANSI color or VT100 support.                 |
| ANSI  | Terminal supports the common ANSI color codes. Supporting blink and underline is optional. |
| VT100 | Terminal supports most VT100 codes and ANSI color codes.     |
| XTERM | Terminal supports all VT100 and ANSI color codes, 256 colors, mouse tracking, and all commonly used xterm console codes. |

If 256 color detection for non MTTS compliant servers is a must it's an option to report "ANSI-256COLOR", "VT100-256COLOR", or "XTERM-256COLOR". If -TRUECOLOR is used to indicate truecolor support the client should also support 256 colors.

## Mud Terminal Type Standard
On the third TTYPE SEND request the client should return MTTS followed by a bitvector. The bit values and their names are defined below.
| Bit-Value | Keyword | Description                                                  |
| ----- | ------------------ | ---------------------------------------------------- |
|    1  | "ANSI"             | Client supports all common ANSI color codes. |
|    2  | "VT100"            | Client supports all common VT100 codes. |
|    4  | "UTF-8"            | Client is using UTF-8 character encoding.
|    8  | "256 COLORS"       | Client supports all 256 color codes.
|   16  | "MOUSE TRACKING"   | Client supports xterm mouse tracking.
|   32  | "OSC COLOR PALETTE"| Client supports OSC and the OSC color palette.
|   64  | "SCREEN READER"    | Client is using a screen reader.
|  128  | "PROXY"            | Client is a proxy allowing different users to connect from the same IP address.
|  256  | "TRUECOLOR"        | Client supports truecolor codes using semicolon notation.
|  512  | "MNES"             | Client supports the Mud New Environment Standard for information exchange.
| 1024  | "MSLP"             | Client supports the Mud Server Link Protocol for clickable link handling.
| 2048  | "SSL"              | Client supports SSL for data encryption, preferably TLS 1.3 or higher.

The client should add up the numbers of all supported terminal capabilities and print it as ASCII in decimal notation. In the case that a client supports ANSI, UTF-8, as well as 256 COLORS, it should respond with "MTTS 13", which is the sum of 1, 4, and 8. The reporting of UTF-8 should be implemented as a user setting, unless the client is certain that a Unicode font is being used.

## Cycling
RFC1091 allows for cycling through a list of terminal types in order to select one. Implementing this behavior is optional, but the client must properly report the end of the cycle.
At the fourth TTYPE SEND requests the client should repeat the previous response, reporting MTTS followed by a bitvector. Receiving the same terminal type twice indicates to the server that the end of the list of available terminal types has been reached. If the server sends additional requests the client can either continue to respond with ``MTTS <bitvector>``, reset its cycling state to the initial state and start over, or ignore the request.
If the server sends IAC DONT TTYPE the client's cycling state should be reset to the initial state, as if the client just connected to the server. This behavior allows recovering from a copyover.

## Proxies
It's suggested for proxies adding MTTS support to also implement the 
NEW-ENVIRON telnet option as per RFC1572. Using the NEW-ENVIRON option the 
server can send: ``IAC SB NEW_ENVIRON SEND VAR "IPADDRESS" IAC SE``. When 
receiving this send request the proxy should respond with: ``IAC SB NEW_ENVIRON IS VAR "IPADDRESS" VALUE "<user's ip address>" IAC SE``. 
See the MNES specification for more information.
The quote characters mean that the encased word is a string, the quotes themselves should not be send.

