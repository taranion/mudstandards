---
sidebar_label: Intermud  (CD Intermud)
description: The one that started it
---
# Intermud 1 / CD InterMUD 

:::note
**Source**: [http://mud.stack.nl/intermud/cd.protocol.html](https://web.archive.org/web/19970712120618/http://mud.stack.nl/intermud/cd.protocol.html)
:::

:::note[Comment from source in 1999]
The original intermud protocol seems to be the CD-version (designed by people from Chalmers Datorförening). Here (small) UDP messages are send from one mud to another. Some messages, like channel messages, are send to all known muds. You can extend the list off known muds by querying other muds for the list they know, or by adding muds when you receive a message from them.

Well, doubt that this was the first intermud implementation. But I have yet to find other references/implementations... This version dates back from August 1992 and was implemented in CD.01.55 (driver version). It seems to have been much driven by Commander@Genesis (Johan Andersson). 
:::

This file gives a description of how CDlibs and CDdrivers can communicate with each other and the protocol used.

Intermud communication is done via udp packages, contrary to players connected to the mud whose communication goes through tcp. This has two consequences:

- Intermud communication will be unreliable. This is the nature of udp.
- Only two filedescriptors are used for all intermud communication, not one / mud.

The entire protocol is defined in LPC and the gamedriver only has support for sending and receiving, i.e:

- Send a message to a specific host:port
- Notify the master object when it has received a message from a host.

Note that the 'port' is a special 'udp reception port', not the usual port that players use to connect to the mud. There is a #define in the GD's config.h that tells what udp-port to use for reception. There is also a '-u' flag that can be given to the GD when started, to denote udp-port.

## Mudlib support

The GD allows only the master object to send udp messages. There is a function 'send_udp_message' in /secure/master that can be called to send a message. Normally only a specific 'udp manager' is allowed by /secure/master to send messages. What object to use as UDP_MANAGER is defined in  /config/sys/local.h

There is a file /sys/global/udp that can be used as udp manager or inherited by the object used as such. /sys/global/udp holds all routines necessary for the implementation of the basic intermud protocol.

Among the more important is the keeping track of the known muds. There is a possibility for the manager to store the list of known muds in /secure/master so as to keep the list over a reboot. There is commands for the managing of this, 'mudlist', 'storemuds', see help on these.

## Protocol design

- Every message sent is printable strings.
- Every message consists of a 'command' and 'parameters'. Paramaters can have subparameters. 

The typical message string looks like:
```
@@@command||Param1:Paramvalue1||Param2:|Subparam2.1|Subparam2.2@@@
```
or in a slightly more formal description:

```
	message 		::= '@@@' command parameterlist '@@@'

	command 		::= Alfanumword

	parameterlist 		::= /* empty */
			  	    | '||' tag ':' value parameterlist

	tag  			::= Alfanumword

	value 			::= String
				    | subparameterlist

	subparameterlist 	::= /* Empty */
			      	    | '|' tag ':' value subparameterlist
```

Where Alfanumword is one word of letters and digits, and 'String' is  any printable text not containing the '|', '||' or '@@@' letter combinations.

The motivation for this design is the simple translation of a message into a mapping of parameters. This is very simple to do by exploding the message first on '||', separating the parameters and then to explode on '|' for each parametervalue, separating the subparameters. 

The parameterlist is then put into a mapping using the tags as index. The values of the mapping can then be either a string or another mapping holding the subparameters.

The extendibility of this protocol is evident. Both new commands and new parameters to old commands can be added without damaging the backwards compatibility of the new protocol visavi older versions of the protocol.

## Current protocol

Only a few commands has been implemented so far. Their names are defined in the file /sys/udp.h

The commands, i.e. the 'Alfanumwords' for each command are: 

### "startup"

  This command is sent to the 'mudlist server' when the mud starts. The 'mudlist server' is defined in /secure/std.h Typical parameter tags are:  "NAME"The name of the mud, for example "Genesis" "VERSION"The version of the GD, for example "CD.01.57" "MUDLIB"The version of the mudlib, ex. "CD.00.25" "HOST"The hostname running the mud "PORT"The portnumber for players to connect to "PORTUDP"The portnumber to receive udp messages on "TIME"The localtime of startup, UNIX ctime-format

### "shutdown"

  This command is sent to the mudlist server when the mud shuts down. Typical parameters are the same as for startup.

### "ping_q"

  This command is sent from one mud to another to request information. Typical parameters sent are:  "NAME"The name of the mud, for example "Genesis" "PORTUDP"The portnumber to receive udp messages on

### "ping_a"

  This command is sent as an answer to a 'ping query' command. The parameters are the same as in the startup command.

### "mudlist_q"

  This command is sent from one mud to another to get info on what muds it knows about. Parameters are the same as for a 'ping_q' command.

### "mudlist_a"

  This command is sent as an answer to a 'mudlist query' command. The parameters are simple numeric indexes, one for each known mud. For each such index is then given a subparameterlist holding these typical 'subtags':  "NAME"The name of the mud, for example "Genesis" "HOST"The hostname running the mud "HOSTADDRESS"The IP adress used to talk with the mud "PORT"The portnumber for players to connect to "PORTUDP"The portnumber to receive udp messages on  Typical example: `		@@@mudlist_a 		||1:|NAME:GENESIS|HOST:milou 			|HOSTADRESS:129.16.79.12 			|PORT:2000|PORTUDP:2500 		||2:|NAME:Othermud|HOST:other 			|HOSTADRESS:127.0.0.1 			|PORT:3000|PORTUDP:4500 	@@@ `

### "warning"

  This is sent as to warn another mud that someone is trying to send messages using the same mudname but from another host. Typical parameters:  "MSG"A message containing the warning "FAKEHOST"IP address of the falsifier

### "gwizmsg"

  This is sent from one mud to another holding a message to present on the 'global wizline'. Typical parameters are:  "NAME"Name of the mud where the message is from. "WIZNAME"Name of the wizards sending the message "GWIZ"The actual message text as given by the wizard

## Future extensions

There are numerous possibilities for new commands in the protocol. Some  examples of what could be added are commands to handle:

- Remote who, what wizards/players are logged in on a specific mud
- Remote finger - Give special information on a wizard/player
- Remote tell - Direct communication to a specific wizard/player
- Remote control - Control an NPC in a foreign mud.
- Intermud mail - Send mail to someone at somemud
- Simple mud to mud filetransfer.

Further in the future are:

- Capabilities to transfer source code between muds. Perhaps having a special 'foreign mud' domain, where each 'wizard' is another mud.
- 'Online' updates of the distributed mudlib code.

## Extension design

The standard udp manager '/sys/global/udp' has been written so as to simplify adding new commands. Essentially what is needed is to write a separate file defining the command and then inheriting that into the object used as udp manager on the specific mud.

This also makes it easier for each mud to make extensions to the protocol and then distribute each extension as a separate file. Each such file intended to be 'multiply inherited'.

Therefore it is recommended to make your own 'udp manager' in which you inherit the standard and then also inherit each of your local extensions. This new udp manager should then replace the old as defined in /config/sys/local.h

The key to this, is this piece of code in /sys/global/udp.c:

```
  /* 
   * Execute the given commands
   */
  int
  execure_udp_command(string cmd, mapping params)
  {
      switch(cmd)
      {	
      case UDP_STARTUP:
      case UDP_SHUTDOWN:
      case UDP_PING_Q:
      case UDP_PING_A:
      case UDP_MUDLIST_Q:
      case UDP_MUDLIST_A:
      case UDP_WARNING:
      case UDP_GWIZMSG:
	  return call_other(this_object(), cmd, params);
      default:
	  return 0;
      }
  }
```

Note that the command is called as a function. What is needed in your separate file is simply to define a function with the same name as the new command.

You also need to redefine the 'execute_udp_command' above to include your own command additions in the switch.