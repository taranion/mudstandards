---
sidebar_label: MSDP (Option 69)
---
**Source**: [Mudhalla](https://mudhalla.net/tintin/protocols/msdp/)

MUD servers often want to send additional data to a MUD client that doesn't necessarily need to be displayed, as well as needing a way to identify clients that support out-of-band data.

The MUD Server Data Protocol seeks to address these issues by providing a transparant and straight forward out-of-band protocol for MUD servers to send variables and their values to MUD clients that can be both generic, and specific for the MUD in question. Using generic variables will allow generic client interface scripts that work on all MSDP MUDs.

This document provides a technical description of the MSDP protocol.

# MUD Server Data Protocol (MSDP)

MSDP is implemented as a Telnet option RFC854, RFC855. The server and client negotiate the use of MSDP as they would any other telnet option. Once agreement has been reached on the use of the option, option sub-negotiation is used to exchange information between the server and client.

## Server Commands
```
IAC WILL MSDP    Indicates the server wants to enable MSDP.
IAC WONT MSDP    Indicates the server wants to disable MSDP.
```
## Client Commands
```
IAC DO   MSDP    Indicates the client accepts MSDP sub-negotiations.
IAC DONT MSDP    Indicates the client refuses MSDP sub-negotiations.
```
## Handshake
When a client connects to an MSDP enabled server the server should send IAC WILL MSDP. The client should respond with either IAC DO MSDP or IAC DONT MSDP. Once the server receives IAC DO MSDP both the client and the server can send MSDP sub-negotiations.

The client should never initiate a negotiation, if this happens however the server should abide by the state change. To avoid trigger loops the server should not respond to negotiations from the client, unless it correctly implements the Q method in RFC 1143.

## Disabling MSDP
When a typical MUD server performs a copyover it loses all previously exchanged MSDP data. If this is the case, before the actual copyover, the MUD server should send IAC WONT MSDP, the client in turn should fully disable MSDP. After the copyover has finished the server and client behave as if the client has just connected, so the server should send IAC WILL MSDP.

When a typical MUD client loses its link and reconnects it loses all previously exchanged MSDP data. The server should reset its MSDP state and re-negotiate MSDP whenever a client reconnects.

## MSDP definitions
```
MSDP              69

MSDP_VAR           1
MSDP_VAL           2

MSDP_TABLE_OPEN    3
MSDP_TABLE_CLOSE   4

MSDP_ARRAY_OPEN    5
MSDP_ARRAY_CLOSE   6
```

## Example MSDP handshake
```
server - IAC WILL MSDP
client - IAC   DO MSDP
client - IAC   SB MSDP MSDP_VAR | LIST|  MSDP_VAL | COMMANDS|  IAC SE
server - IAC   SB MSDP MSDP_VAR | COMMANDS|  MSDP_VAL MSDP_ARRAY_OPEN MSDP_VAL | LIST|  MSDP_VAL | REPORT|  MSDP_VAL | SEND|  MSDP_ARRAY_CLOSE IAC SE
client - IAC   SB MSDP MSDP_VAR | LIST|  MSDP_VAL | REPORTABLE_VARIABLES|  IAC SE
server - IAC   SB MSDP MSDP_VAR | REPORTABLE_VARIABLES|  MSDP_VAL | HINT|  IAC SE
client - IAC   SB MSDP MSDP_VAR | SEND|  MSDP_VAL | HINT|  IAC SE
server - IAC   SB MSDP MSDP_VAR | HINT|  MSDP_VAL | THE GAME|  IAC SE
```
The quote characters mean that the encased word is a string, the quotes themselves should not be send.

### Variables and Values
Variables are send as a typical telnet sub-negotiation having the format: ``IAC SB MSDP MSDP_VAR <VARIABLE> MSDP_VAL <VALUE> IAC SE``. For ease of parsing, variables and values cannot contain the NUL, MSDP_VAL, MSDP_VAR, MSDP_TABLE_OPEN, MSDP_TABLE_CLOSE, MSDP_ARRAY_OPEN, MSDP_ARRAY_CLOSE, or IAC byte. For example:
```
IAC SB MSDP MSDP_VAR | SEND|  MSDP_VAL | HEALTH|  IAC SE
```
The quote characters mean that the encased word is a string, the quotes themselves should not be send.

### Tables
Sometimes it's useful to send data as a table, which can be done in MSDP using MSDP_TABLE_OPEN and MSDP_TABLE_CLOSE after MSDP_VAL, and nest variables and values inside the MSDP_TABLE_OPEN and MSDP_TABLE_CLOSE arguments. Tables are called objects in the JSON standard, they're also known as maps, dictionaries, and associative arrays.
A ROOM data table in MSDP would look like:
```
IAC SB MSDP 
MSDP_VAR | ROOM|  MSDP_VAL MSDP_TABLE_OPEN 
 MSDP_VAR | VNUM|  MSDP_VAL | 6008|  
 MSDP_VAR | NAME|  MSDP_VAL | The forest clearing|  
 MSDP_VAR | AREA|  MSDP_VAL | Haon Dor|  
 MSDP_VAR | TERRAIN|  MSDP_VAL | forest|  
 MSDP_VAR | EXITS|  MSDP_VAL MSDP_TABLE_OPEN 
  MSDP_VAR | n|  MSDP_VAL | 6011|  
  MSDP_VAR | e|  MSDP_VAL | 6007|  
 MSDP_TABLE_CLOSE 
MSDP_TABLE_CLOSE 
IAC SE
```
The quote characters mean that the encased word is a string, the quotes themselves should not be send.

### Arrays
Sometimes it's useful to send data as an array, which can be done in MSDP using MSDP_ARRAY_OPEN and MSDP_ARRAY_CLOSE after MSDP_VAL, and nest values inside the MSDP_ARRAY_OPEN and MSDP_ARRAY_CLOSE arguments, with each value preceded by an MSDP_VAL argument.
An array in MSDP would look like:
```
IAC SB MSDP MSDP_VAR | REPORTABLE_VARIABLES|  MSDP_VAL MSDP_ARRAY_OPEN MSDP_VAL | HEALTH|  MSDP_VAL | HEALTH_MAX|  MSDP_VAL | MANA|  MSDP_VAL | MANA_MAX|  MSDP_ARRAY_CLOSE IAC SE
```
The quote characters mean that the encased word is a string, the quotes themselves should not be send.

### Lists
To minimize the implementation burden for servers, the client is expected to send command arguments as simple value reassignments. This looks as following:
```
IAC SB MSDP MSDP_VAR | REPORT|  MSDP_VAL | HEALTH|  MSDP_VAL | HEALTH_MAX|  MSDP_VAL | MANA|  MSDP_VAL | MANA_MAX|  IAC SE
```
The quote characters mean that the encased word is a string, the quotes themselves should not be send.
When responding to the client, the server should use a proper array if applicable.

## Reportable MSDP Variables

These variables are mere suggestions for MUDs wanting to implement MSDP. By using these reportable variables it'll be easier to create MSDP scripts that need little or no modification to work across MUDs. If you create your own set of variables in addition to these, it's suggested to create an extended online specification that describes your variables and their behavior. The SPECIFICATION variable can be used to link people to the web page.

### General

| Name           | Description                                                  |
| -------------- | ------------------------------------------------------------ |
| ACCOUNT_NAME   | Name of the player account.                                  |
| CHARACTER_NAME | Name of the player account.                                  |
| SERVER_ID      | Name of the MUD, or an otherwise unique ID.                  |
| SERVER_TIME|    | The time on the server using either military or civilian time. |
| SPECIFICATION  | URL to the MUD's online MSDP specification, if any.          |

### Character
| Name           | Description                                                  |
| -------------- | ------------------------------------------------------------ |
| AFFECTS        | Current affects in array format.                            |
| ALIGNMENT      | Current alignment.                                 |
| EXPERIENCE     | Current total experience points. Use 0-100 for percentages.               |
| EXPERIENCE_MAX | Current maximum experience points. Use 100 for percentages. |
| EXPERIENCE_TNL | Current total experience points until Next Level. Use 0-100 for percentages.      |
| HEALTH|                Current health points.|
| HEALTH_MAX|            Current maximum health points.|
| LEVEL|                 Current level.|
| MANA|                  Current mana points.|
| MANA_MAX|              Current maximum mana points.|
| MONEY|                 Current amount of money.|
| MOVEMENT|              Current movement points.|
| MOVEMENT_MAX|          Current maximum movement points.|

### Combat
| Name           | Description                                                  |
| -------------- | ------------------------------------------------------------ |
| OPPONENT_LEVEL | Level of opponent.
| OPPONENT_HEALTH| Current health points of opponent. Use 0-100 for percentages.
| OPPONENT_HEALTH_MAX|   Current maximum health points of opponent. Use 100 for percentages.
| OPPONENT_NAME  | Name of opponent.
| OPPONENT_STRENGTH|     Relative strength of opponent, like the consider mud command.

### Mapping
Indentation indicates the variable is nested within the parent variable using a table.

| Name           | Description                                                  |
| -------------- | ------------------------------------------------------------ |
| ROOM           | |
| -  VNUM   |                A number uniquely identifying the room.|
| -  NAME   |                The name of the room.|
| -  AREA|                The area the room is in.|
| -  COORDS| |
| --    X|                 The X coordinate of the room.|
| --    Y|                 The Y coordinate of the room.|
| --    Z|                 The Z coordinate of the room.|
| -  TERRAIN|             The terrain type of the room. Forest, Ocean, etc.|
| EXITS|               Nested abbreviated exit directions (n, e, w, etc) and corresponding destination VNUMs.|

### World
| Name           | Description                                                  |
| -------------- | ------------------------------------------------------------ |
| WORLD_TIME|            The in game time on the MUD using either military or civilian time.|


## Configurable MSDP Variables
Configurable variables are variables on the server that can be altered by the client. Implementing configurable variable support is optional.

### General
| Name           | Description                                                  |
| -------------- | ------------------------------------------------------------ |
| CLIENT_NAME    | Name of the MUD client.     | 
| CLIENT_VERSION | Version of the MUD client.  | 
| PLUGIN_ID|      | Unique ID of the MSDP plugin/script. | 

## Generic MSDP Commands
These are variables that behave as commands for more complex server - client interaction. Implementing the following 5 MSDP commands correctly is essential.

### LIST
The LIST command can be used by either side, but should typically be used by the client. To request a list of commands use IAC SB MSDP MSDP_VAR | LIST|  MSDP_VAL | COMMANDS|  IAC SE. The receiver in turn should send back the | COMMANDS|  variable with an array of values containing supported commands, for example: IAC SB MSDP MSDP_VAR | COMMANDS|  MSDP_VAL MSDP_ARRAY_OPEN MSDP_VAL | LIST|  MSDP_VAL | REPORTABLE_VARIABLES|  MSDP_ARRAY_CLOSE IAC SE. The following list options are suggested:

| Name           | Description                                                  |
| -------------- | ------------------------------------------------------------ |
| COMMANDS       | Request an array of commands supported by the server.
| LISTS          | Request an array of lists supported by the server.
| CONFIGURABLE_VARIABLES|  Request an array of variables the client can configure.
| REPORTABLE_VARIABLES|    Request an array of variables the server will report.
| REPORTED_VARIABLES|      Request an array of variables currently being reported.
| SENDABLE_VARIABLES|      Request an array of variables the server will send.

```
client - IAC SB MSDP MSDP_VAR "LIST" MSDP_VAL "CONFIGURABLE_VARIABLES" IAC SE
server - IAC SB MSDP MSDP_VAR "CONFIGURABLE_VARIABLES" MSDP_VAL MSDP_ARRAY_OPEN MSDP_VAL "UTF_8" MSDP_VAL "XTERM_256_COLORS" MSDP_ARRAY_CLOSE IAC SE
client - IAC SB MSDP MSDP_VAR "UTF_8" MSDP_VAL "0" MSDP_VAR "XTERM_256_COLORS" MSDP_VAL "1" IAC SE
```

### REPORT
The REPORT command can be used by either side, but should typically be used by the client. After the client has received a list of reportable variables, or already knows which variables are supported, it can request the server to start reporting those variables. When receiving a REPORT request the server should send the requested variables to the client, and re-send each individual variable whenever it changes. Instead of sending MSDP_VAR "REPORT" MSDP_VAL "HEALTH" MSDP_VAR "REPORT" MSDP_VAL "HEALTH_MAX" it is allowed to string values together for command-like variables, in this case: MSDP_VAR "REPORT" MSDP_VAL "HEALTH" MSDP_VAL "HEALTH_MAX".
```
client - IAC SB MSDP MSDP_VAR "LIST" MSDP_VAL "REPORTABLE_VARIABLES" IAC SE
server - IAC SB MSDP MSDP_VAR "REPORTABLE_VARIABLES" MSDP_VAL MSDP_ARRAY_OPEN MSDP_VAL "MUD_TIME" MSDP_VAL "SERVER_TIME" MSDP_ARRAY_CLOSE IAC SE
client - IAC SB MSDP MSDP_VAR "REPORT" MSDP_VAL "MUD_TIME" IAC SE
server - IAC SB MSDP MSDP_VAR "MUD_TIME" MSDP_VAL "14:00" IAC SE
server - IAC SB MSDP MSDP_VAR "MUD_TIME" MSDP_VAL "15:00" IAC SE
```

### RESET
The RESET command works like the LIST command, and can be used to reset groups of variables to their initial state. Most commonly RESET will be called with REPORTABLE_VARIABLES or REPORTED_VARIABLES as the argument, though any LIST option can be used.
```
client - IAC SB MSDP MSDP_VAR "RESET" MSDP_VAL "CHESS_MINIGAME" IAC SE
```


### SEND
The SEND command can be used by either side, but should typically be used by the client. After the client has received a list of variables, or otherwise knows which variables exist, it can request the server to send those variables and their values with the SEND command. The value of the SEND command should be a list of variables the client wants returned.
```
client - IAC SB MSDP MSDP_VAR "SEND" MSDP_VAL "AREA_NAME" MSDP_VAL "ROOM_NAME" IAC SE
server - IAC SB MSDP MSDP_VAR "AREA_NAME" MSDP_VAL "Tower of Entropy" MSDP_VAR "ROOM_NAME" MSDP_VAL "Tower Pinnacle" IAC SE
```


### UNREPORT
The UNREPORT command is used to remove the report status of variables after the use of the REPORT command.

```
client - IAC SB MSDP MSDP_VAR "UNREPORT" MSDP_VAL "MUD_TIME" MSDP_VAL "NEWBIE_CHANNEL" IAC SE
```
