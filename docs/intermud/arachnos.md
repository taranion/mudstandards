---
sidebar_label: Arachnos
description: A intermud messaging via MSDP
---
# Aranos Intermud Network

Source: https://tintin.mudhalla.net/protocols/msdp/arachnos/index.php

Arachnos is an Intermud standard for sending messages between MUD servers utilizing MSDP. Instead of the classic approach where each MUD server connects to a central server, a spider is used which connects to each MUD using their existing connection and telnet handler. The MSDP data protocol is used to transmit structured data between the spider and MUD server.

Advantages of this approach are that if the spider's server goes down the spider is easily ran from somewhere else, and multiple spiders can co-exist providing different or enhanced services.

## Implementing Arachnos Intermud Support
In order to announce Arachnos support the MUD server needs to implement MSSP and set the INTERMUD variable to Arachnos. See the MSSP specification for more information.

### Identifying Spiders
When a spider connects and MSDP is negotiated it should send the ARACHNOS_ID variable holding a value between 1 and 2000000000. The ARACHNOS_ID value should remain the same for each MUD, but differ between MUDs, serving as both a form of identification and as a password. The MUD can use ARACHNOS_ID to differentiate between trusted and untrusted spiders, or ignore it, relying on existing ban controls to deal with disruptive spiders.

### Arachnos developer channel
The developer channel is one of the two default channels defined in this standard. After the spider connects it should ask for the ARACHNOS_DEVEL variable to be reported. The MUD in turn should create a communication command that generates an ARACHNOS_DEVEL update when used, the spider in turn will echo the message to all MUDs it is connected to, including the originating MUD.

The ARACHNOS_DEVEL variable update sent from the MUD to the spider should contain a table containing the MSG_USER and MSG_BODY variable, this would look as following.
```
IAC SB MSDP VAR "ARACHNOS_DEVEL" VAL TABLE_OPEN VAR "MSG_USER" VAL "Bubba" VAR "MSG_BODY" VAL "Hello Arachnos!" TABLE_CLOSE IAC SE
```
This message will be forwarded by the spider to each MUD server using (once again) the ARACHNOS_DEVEL variable containing a table containing the MUD_NAME, MUD_HOST, MUD_PORT, MSG_USER, and MSG_BODY variables. The MUD server should parse the table and display the message to developers.
```
IAC SB MSDP VAR "ARACHNOS_DEVEL" VAL TABLE_OPEN VAR "MSG_USER" VAL "Bubba" VAR "MSG_BODY" VAL "Hello Arachnos!" VAR "MUD_NAME" VAL "MUDhalla" VAR "MUD_HOST" VAL "mudhalla.net" VAR "MUD_PORT" VAL "42" TABLE_CLOSE IAC SE
```

### Arachnos chat channel
The chat channel behaves identically to the developer channel described above, but is instead intended for players. The spider will ask for the ARACHNOS_CHAT variable to be reported, and respond to ARACHNOS_CHAT updates.

### Arachnos Mudlist
The mudlist variable is sent by the spider to allow MUDs to create a list of connected servers. MSSP is used to retrieve the name, uptime, and number of players of each MUD. At regular intervals the spider will retrieve updated information for a single MUD and broadcast it to all connected MUDs. MUDs in turn can add new mudlist entries to a list, and update existing entries.

The name of the mudlist variable is ARACHNOS_MUDLIST which contains a table which contains the MUD_NAME, MUD_HOST, MUD_PORT, MUD_UPTIME, and MUD_PLAYERS variables. An ARACHNOS_MUDLIST variable update would look as following:
```
IAC SB MSDP VAR "ARACHNOS_MUDLIST" VAL TABLE_OPEN VAR "MUD_NAME" VAL "Bubbledum" VAR "MUD_HOST" VAL "bubbles.com" VAR "MUD_PORT" VAL "23" VAR "MUD_UPTIME" VAL "1311624730" VAR "MUD_PLAYERS" VAL "36" TABLE_CLOSE IAC SE
```
