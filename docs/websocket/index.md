---
sidebar_label: WebSocket for MUDs
description: Suggestions how to standardize WebSocket usage for MUDs
---
# Using RFC 6455 WebSocket for MUDs

Being able to connect to a game using a WebSocket instead of a classic Telnet connection is a plus for web or mobile clients. But what exactly gets transmitted over the socket to the application layer and how (as a binary or text frame)? 

## The Current Situation

Today (2025) the majority of servers and codebases use one of the following approaches how to use a WebSocket:

1. Tunnel the full telnet stream (incl. all options and user data) through the WebSocket. Often just making a local connection to the telnet socket and pass data in both directions. 
2. Send custom JSON structures to either contain text (game output, user input) or out-of-band (OOB) messages. These JSON structures are often specific to a codebase or even a single game and used to power their custom web clients.
3. An in-between approach, where just the ASCII/ANSI stream is transmitted.

All approaches have their pros and cons, but the variety prevents writing a generic WebSocket client. The goal of this document is to come up with a standardization that allows all ways to coexist and yet remain usable.

## Subprotocols to the rescue

In the opending handshake of a WS connection, the client use optional the `Sec-WebSocket-Protocol` header, to provide a comma-separated list of protocols the client understands.

> ```
> The |Sec-WebSocket-Protocol| request-header field can be used to indicate what 
> subprotocols (application-level protocols layered over the WebSocket Protocol) 
> are acceptable to the client. The server selects one or none of the acceptable
> protocols and echoes that value in its handshake to indicate that it has 
> selected that protocol.
> ```

In the response, when switching to WebSocket communication, the server than sends a `Sec-WebSocket-Protocol` header, with the protocol the server selected - or rejects the connection, if the negotiation fails.
Read more about it in [RFC 6455](https://datatracker.ietf.org/doc/html/rfc6455#section-1.9).

## MUD WebSocket Protocols

This document suggests the following protocols:

| Protocol*                   | OPCODEs    | Description                                                  |
| --------------------------- | ---------- | ------------------------------------------------------------ |
| `telnet.mudstandards.org`   | 1 (BINARY) | The complete telnet stream is packaged in BINARY frames. All telnet options are transmitted this way too. |
| `terminal.mudstandards.org` | 1 (BINARY) | BINARY frames contain input/output and ANSI control codes. Encoded as UTF-8 |
| `gmcp.mudstandards.org`     | 0 + 1      | BINARY frames do contain regular ANSI in- and output. TEXT frames contain UTF-8 encoded GMCP commands |
| `json.mudstandards.org`     | 0 + 1      | BINARY frames do contain regular ANSI in- and output. TEXT frames contain the [specific JSON payload](#json) defined in this document. |
| `divstream.mudstandards.org`| 1 (BINARY) | BINARY frames do contain HTML DIV elements to render at the bottom of the scrolling area. |
| e.g. *myprotocol.mydomain*  | ?          | Any codebase custom protocol a client supports               |

*The RFC requests that protocol names are build upon a domain name, to prevent name collisions. 

## <a name="json"></a>The JSON Format

The goal of this format is to express support for a large list of protocols. 

```json
{
	"proto": "<string>",
    "id": "<string>",      # Optional
    "data": "<string>"

}
```

| Field | TYPE   | Required | Description                                                  |
| ----- | ------ | -------- | ------------------------------------------------------------ |
| proto | string | yes      | What protocol is in the data                                 |
| id    | string | no       | Something to further specify the content - e.g. a command name |
| data  | string | yes      | Payload to be decoded into whatever the protocol expects.    |

**Examples**

```json
{
	"proto":	"gmcp",
	"id": "Core.Hello",
	"data": "{\"Client\": \"Mudterm\", \"Version\": \"0.2.0\"}"
}
```

```json
{
	"proto":	"telnet_31",
    "id": "NAWS",
	"data": "...base64 encoded NAWS frame.."
}
```

```json
{
	"proto": "coffeemud",
	"id": "login",
	"data": "..."
}
```

## Custom MUD protocols

Servers that already do send their own JSON structure over a websocket today can continue doing so. All they need to do is add a protocol name (e.g. `v1.evennia.com` ) to the protocol header on the client side and only accept incoming connections with that protocol among the list of supported protocols. 

## MUD WebSocket Extensions

WebSocket "Extension" are a mechanism to either apply functions to the payload data (e.g. compression), declare the first X bytes of the payload as "Extension Data" or agree upon how certain frame opcodes are to be interpreted.

In theory extensions would be suitable to multiplex a WebSocket payload into several protocols, but since support for custom extensions is not widespread among WebSocket implementation libraries, it does not seem feasible.

## Restrictions

- The websocket, once established, has no means to change negotiated protocols or extensions and therefore can not be used in a similar fashion like DO/WILL/DONT/WONT of telnet. It is up to an application layer protocol to provide that.
- The protocol `telnet.mudstandards.org` usually means that the server passes data to and from a local telnet socket. This hides the original source IP from the MUD. The server might intercept MNES subnegotiation and insert the `IPADDRESS` data, but that is a lot of effort