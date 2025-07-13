---
sidebar_label: core
---
# The ``Core`` package

This package should be supported by any GMCP client

## Core.Hello

Needs to be the first message that the client sends, used to identify the client
````json
Core.Hello {
    "Client": "Mudlet",
    "Version":"3.0.0"
}
````
- **Client**
  (*Mandatory*) Name of the client
- **Version**
  (*Mandatory*) Client version

## Core.Goodbye
Sent by server immediately before terminating a connection
```json
Core.Goodbye "Goodbye, adventurer"
```
Message body is a string to be shown to the user - it can explain the reason for the disconnect.

## Core.KeepAlive
Causes the server to reset the timeout for the logged character, no message body
```json
Core.KeepAlive 
```

## Core.Ping

Sent either by client or server.
The client starts the process by sending a ''Core.Ping'' and memorizes the timecode.
The server responds to the request by replying with ``Core.Ping`` without a body.
The client when it receives the response, checks how many milliseconds have passed since the request has been sent. 

In regular intervals the client repeats the process. The messages sent by the client get the last observed round trip time as a parameter.

````json
Core.Ping <round_trip_time>
````
- Round trip time
  (*Optional*) Only present when sent by client. Contains the roundtrip time in milliseconds from the last interval. Encoded as a number.
  

## Core.Supports.Set

Sent by the client. Notifies the server about packages supported by the client
If another Core.Supports.*** package has been received earlier, the list is deleted and replaced with the new one.
Most client implementations will only need to send Set once and won't need Add/Remove; exceptions are module implementations provided by plug-ins.

````json
Core.Supports.Set [ "Char 1", "Char.Skills 1", "Char.Items 1" ]
````

* Message body is an array of strings, each consisting of the module name and version, separated by space
* Module version is a positive non-zero integer

## Core.Supports.Add

Sent by the client. Similar to Set, but appends the supported module list to the one sent earlier
If no list was sent yet, the behaviour is identical to Set
If the list includes module names that were already included earlier, the new version number takes precedence over the previously sent one, even if the newly sent number is lower.

````json
Core.Supports.Add [ "Char 1", "Char.Skills 1", "Char.Items 1" ]
````

## Core.Supports.Remove

Sent by the client. Removes specified modules from the list of supported modules

````json
Core.Supports.Remove [ "Char", "Char.Skills", "Char.Items" ]
````
No version numbers included.
