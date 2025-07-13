---
sidebar_label: MNES (Option 39)
description: Allows a client to inform the server of it's capabilities.
---
# MUD NEW-ENVIRON Standard

MUD servers often want to know the various capabilities of a Mud Client. While various methods exist there is no consistent nor guaranteed way to do so.
The Mud Terminal Type Standard seeks to address these issues by providing a transparant and straight forward standard for Mud Clients to communicate their capabilities. It does so by extending and standardizing RFC1091 which describes the Telnet Terminal-Type Option.
MTTS is limited in scope by design. MNES (Mud New Environment Standard) seeks to supplement MTTS by providing a straightforward way to use the NEW-ENVIRON telnet option to exchange and update various client and server settings.

## The NEW-ENVIRON Protocol

NEW-ENVIRON is implemented as a Telnet option RFC854, RFC855. The server and client negotiate the use of the NEW-ENVIRON option as they would any other telnet option as detailed in RFC1572. Once agreement has been reached on the use of the option, option sub-negotiation is used to exchange information between the server and client.

### Server Commands
IAC DO   NEW-ENVIRON    Indicates the server wants to exchange NEW-ENVIRON variables.
IAC DONT NEW-ENVIRON    Indicates the server doesn't want to exchange NEW-ENVIRON variables.

### Client Commands
IAC WILL NEW-ENVIRON    Indicates the client will exchange NEW-ENVIRON variables.
IAC WONT NEW-ENVIRON    Indicates the client will not exchange NEW-ENVIRON variables.

### Handshake
When a client connects to a NEW-ENVIRON enabled server the server should send IAC DO NEW-ENVIRON. The client should respond with either IAC WILL NEW-ENVIRON or IAC WONT NEW-ENVIRON. Once the server receives IAC WILL NEW-ENVIRON the server can send NEW-ENVIRON sub-negotiations and the client can respond to these.
The client should never initiate a negotiation, in the case this does happen the server should abide by the state change. To avoid trigger loops the server should not respond to negotiations from the client, unless it correctly implements the Q method in RFC 1143.

### Disabling NEW-ENVIRON
When a typical MUD server performs a copyover it may lose all previously exchanged NEW-ENVIRON data. If this is the case, before the actual copyover, the MUD server should send IAC DONT NEW-ENVIRON, the client in turn should fully disable and reset its NEW-ENVIRON state. After the copyover has finished the server and client behave as if the client has just connected, so the server should send IAC DO NEW-ENVIRON.
When a typical MUD client loses its link and reconnects it loses all previously exchanged MSDP data. The server should reset its NEW-ENVIRON state and re-negotiate NEW-ENVIRON whenever a client reconnects.

### NEW-ENVIRON definitions

```
NEW-ENVIRON      39

IS                0
SEND              1
INFO              2

VAR               0
VAL               1
ESC               2
USERVAR           3
```





### Example NEW-ENVIRON handshake

```
server - IAC DO NEW-ENVIRON

client - IAC WILL NEW-ENVIRON

server - IAC   SB NEW-ENVIRON SEND VAR "CLIENT_NAME" VAR "CLIENT_VERSION" IAC SE

client - IAC   SB NEW-ENVIRON IS   VAR "CLIENT_NAME" VAL "TINTIN++" IAC SE

client - IAC   SB NEW-ENVIRON IS   VAR "CLIENT_VERSION" VAL "2.01.7" IAC SE

server - IAC   SB NEW-ENVIRON SEND VAR "CHARSET" IAC SE

client - IAC   SB NEW-ENVIRON IS    VAR "CHARSET" VAL "ASCII" IAC SE
```

The quote characters mean that the encased word is a string, the quotes themselves should not be sent.

### Variables and Values
A server can request the client to send variables with a typical telnet sub-negotiation having the format: ``IAC SB NEW-ENVIRON SEND VAR <VARIABLE> IAC SE``. For ease of parsing, variables should solely exist of upper case letters and underscores, while values cannot contain the VAR, VAL, ESC, USERVAR, or IAC byte (0, 1, 2, 3, 255). A server can request several variables at once. For example:
``IAC SB NEW-ENVIRON IS VAR "CHARSET" VAR "IPADDRESS" IAC SE``
The quote characters mean that the encased word is a string, the quotes themselves should not be sent.

Once the server has requested the client to send a variable the client should answer using the format: ``IAC SB NEW-ENVIRON IS VAR <VARIABLE> VAL <VALUE> IAC SE``. At anytime the client can update the variable using INFO instead of IS.

```
server - IAC DO NEW-ENVIRON

client - IAC WILL NEW-ENVIRON

client - IAC   SB NEW-ENVIRON IS   VAR "CHARSET" VAL "ASCII" IAC SE

client - IAC   SB NEW-ENVIRON INFO VAR "CHARSET" VAL "UTF-8" IAC SE
```

The original NEW-ENVIRON option allows for several ways to request all variables and user variables at once. Servers supporting MNES should either use the IAC SB NEW-ENVIRON SEND VAR IAC SE or IAC SB NEW-ENVIRON SEND IAC SE sequence to request all variables, and clients should handle these sequences properly.

```
server - IAC DO NEW-ENVIRON

client - IAC WILL NEW-ENVIRON

server - IAC   SB NEW-ENVIRON SEND VAR IAC SE

client - IAC   SB NEW-ENVIRON IS   VAR "CLIENT_NAME" VAL "TINTIN++" VAR "CHARSET" VAL "UTF-8" IAC SE
```

## MNES NEW-ENVIRON Variables
These variables are standardized for MUDs wanting to implement NEW-ENVIRON using MNES. The number of variables is intentionally kept to a bare minimum.

| Variable       | Description                                                  |
| -------------- | ------------------------------------------------------------ |
| CHARSET        | Character set.<br />The CHARSET value could be one of ASCII, BIG5, CP437, CP949, CP1251, EUC-KR, GB18030, ISO-8859-1, ISO-8859-2, KOI8-R, UTF-8. It's generally not feasible for a MUD server to force the client to use a specific character set. The MNES CHARSET value should reflect the current character set mode of the client. See man charsets for reference. |
| CLIENT_NAME    | Name of the client.                                          |
| CLIENT_VERSION | Version of the client.                                       |
| IPADDRESS      | The client's IP address.<br />It's suggested for proxies adding MTTS support to also report the client's real IP address. This allows a MUD to ban specific users without having to ban an entire proxy. |
| MTTS           | bitvector<br />The MTTS bitvector that is set by the client should reflect the current MTTS state. See the [MTTS](mtts#mud-terminal-type-standard-1) specification for more information. |
| TERMINAL_TYPE  | name<br />The terminal type that is returned by the client should reflect the terminal type as described in the [MTTS](mtts#mud-client-terminal-type) specification. |

## Links

If you have any comments you can email me at mudclient@gmail.com.



