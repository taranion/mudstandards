---
sidebar_label: Circle Intermud
description: One of the first attempts on inter-mud communication
---
# Circle Intermud

:::note
**Source**: [http://mud.stack.nl/intermud/circle.protocol.html](https://web.archive.org/web/20051229142240/http://mud.stack.nl/intermud/circle.protocol.html)

An other intermud protocol was developed by **Stryker@Tempus** (Chris Austin), and is mostly used on Circle-MUDs. However Merc- and Envy-MUDs are supported as well. The protocol is designed to use a stand-alone daemon for communication with other daemons over UDP and communication with the muddriver using a unix socket. It is quite simple to add support directly to the driver or mudlib though, so there is no reason why LPMUDs shouldn't support this protocol though.

For the rest it was pretty much the same as Intermud-2, however the message fields were indexed by their order in the message rather then by field names. Every message has its own command number which indicates the type of this message. In general these packets are definately shorter than an Intermud-2 message.

Stryker started with version 0.00, however the first only released versions are 0.52-0.56 and finally version 1.0. This was developed in the years 1995-1996 and nothing seems to have been one on this since. Because of this, the protocol was later nicknamed Intermud 1(?!)
:::

## *Introduction*

------

This file gives a description of how Circle, Merc and Envy MUDs can communicate with each other and the protocol used.

Intermud communication is done via udp packages, contrary to players connected to the mud whose communication goes through tcp. This has two consequences:

- Intermud communication will be unreliable. This is the nature of udp.
- Only two filedescriptors are used for all intermud communication, not one / mud.

The protocol is usually implemented in 2 stages. Some functions are defined in the driver, some are defined in an external intermud program. This daemon communicates with the other daemons via udp and with the muddriver via a local unix socket. Some requests from the mud can be handled directly by the daemon, some are relayed to other daemons (and vice versa).

## Protocol design

- Every message sent is printable strings.
- Every message consists of a 'command' and 'parameters'. The set of parameters is fixed for each command.

The typical message string looks like:
```
command|param1|param2|param3|
```
or in a slightly more formal description:

```
	message 		::= command '|' parameterlist

	command 		::= Digit Digit Digit Digit

	parameterlist 		::= /* empty */
			  	    | value '|' parameterlist

	value 			::= String
```

Where digit is any number 0 through 9, and 'String' is any printable text not containing the '|' symbol combinations.

New commands can be added without problems for backwards compatiblilty; whoever it is not possible to add extra parameters to a command.

## Command types

The first digit indicates the type of the command:

- 1xxx

  These are general support commands: startup, ping, mudlist.

- 2xxx

  These indicate the normal request/reply messages: tell, page, who.

- 3xxx

  For messages sent to all MUDs at once: wiz.

- 4xxx

  Driver - local daemon communication commands: mudinfo, stats, debug.

## Current protocol

Only a few commands has been implemented so far. The commands I know are [listed here](https://web.archive.org/web/20051217012619/http://mud.stack.nl/intermud/circle.html).

Here is a list of all services used by the Circle Intermud protocol. This list is NOT yet complete, nor is it completely accurate: I'm still working on this bit...



### startup (1000)

Notify the world that we are alive.  	 

* 1000 The Intermud UDP port 
* The name of our mud 
* The mud's login port (TCP) 
* An integer indicating the type of this mud  (1 = Circle, 2 = Merc, 3 = Envy, 4 = Other) 
* The driver version of this mud Intermud version (usually "1.0") 
* Supported services (usually "F0000000") 

### ping request (1020)

Send out information about ourselves. 

1. 1020 The Intermud UDP port 
2. The name of our mud 
3. The mud's login port (TCP) 
4. An integer indicating the type of this mud  (1 = Circle, 2 = Merc, 3 = Envy, 4 = Other) 
5. The driver version of this mud Intermud version (usually "1.0") 
6. Supported services (usually "F0000000") 

### ping reply (1030)

Send out information about ourselves. 	 

1. 1030 
2. The name of our mud The mud's login port (TCP) #
3. The Intermud UDP port 
4. An integer indicating the type of this mud  (1 = Circle, 2 = Merc, 3 = Envy, 4 = Other) 
5. The driver version of this mud Intermud version (usually "1.0") 
6. Supported services (usually "F0000000") 

### mudlist request (1040)

Retrieve a mudlist. 	 

1. 1040 
2. Reserved for local communication 

### mudlist reply (1050)

Send out information about ourselves. 

1. 1050 
2. The name of a mud 
3. The mud's login port (TCP) 
4. The Intermud UDP port 
5. An integer indicating the type of this mud  (1 = Circle, 2 = Merc, 3 = Envy, 4 = Other) 
6. The driver version of this mud Intermud version (usually "1.0") 
7. Supported services (usually "F0000000") ...  (etc.) 

### dns update (1070)

Send out information about ourselves. (Possibly as a reply to a mudlist.) 

1. 1070 
2. The name of our mud 
3. The mud's login port (TCP) 
4. The Intermud UDP port 
5. The driver version of this mud Intermud version (usually "1.0") 
6. An integer indicating the type of this mud  (1 = Circle, 2 = Merc, 3 = Envy, 4 = Other) 
7. Supported services (usually "F0000000") 

### info (1100)

Send out general information for logging purposes only. 

1. 1100 
2. Reserved for local communication 

### tell request (2000)

Send out information about ourselves. 	 

1. 2000 
2. The player we wish to address 
3. The mud this player is on 
4. The active player 
5. The name of our mud 
6. The message we want to relay 

### tell reply (2010)

Response to a tell or page. 

1. 2010 
2. The wizard to respond to 
3. The mud to send to 
4. "Server" dummy name 
5. The name of our mud 
6. The message, e.g.  
   %s is not currently on-line; 	
   %s is currently mailing or writing; 	
   Your message has been delivered 

### who request (2020)

1. Request login information from another mud. 	 

2. 2020 
3. The mud we want a listing from 
4. The active player 
5. The name of our mud 

### who reply (2030)

Respond to a who request

1. 2030 
2. The mud to respond to 
3. The player to respond to 
4. The name of our mud 
5. Information about an active player  [level] name title ...  (etc.) 

### page (2040)

Send a "page" to another player.

1. 2040 
2. User to page 
3. The mud this user is on 
4. The active player 
5. The name of our mud 
6. The message to send, or NONE for a general page 

### mudinfo request (2050)

Request information on another mud 	 

1. 2050 
2. Reserved for local communication 

### mudinfo reply (2055)

Send out information about ourselves. 	 

1. 2055 
2. Reserved for local communication 

### wiz (3000)

Say something over the intermud wizard channel. 	 

1. 3000 
2. The active player 
3. The name of our mud 
4. The message we wish to relay 

### dns purge (4000)

Force a mudlist purge. 	 

1. 4000 
2. Reserved for local communication 

### reget (4020)

Request a mudlist from the bootmaster. 

1. 4020 
2. Reserved for local communication 

### stats (4040)

Get status information. 	 
4040 
Reserved for local communication 

### mute (4060)

Mute/unmut a complete mud. 	 
4060 
Reserved for local communication 

### debug (4070)

Toggle intermud debugging on/off. 	 
4070 
Reserved for local communication 
