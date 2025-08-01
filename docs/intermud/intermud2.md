---
sidebar_label: Intermud1/2
description: One of the first attempts on inter-mud communication
---
# InterMUD / InterMUD2

:::note
**Source**: [http://mud.stack.nl/intermud/](https://web.archive.org/web/20051229142240/http://mud.stack.nl/intermud/)<br/>
**Source**: [https://mud.ren/threads/170]

The original intermud protocol is the CD UDP-version (designed by people from **Chalmers Datorförening**; a.k.a. the "Genesis" people). Here (small) UDP messages are send from one mud to another. Some messages, like channel messages, are send to all known muds. You can extend the list off known muds by querying other muds for the list they know, or by adding muds when you receive a message from them.

The original protocol has been extended quite a lot over the years, and is currently known as the Intermud 2 protocol. It still is completely backwards compatible with the CD version, however the native CD mudlib doesn't support the new extensions.<br/>
Many distributions are capable of (at least a subset of) this protocol. It was probably first ported for Discworld by **Pinkfish** (David Bennett), nowadays at least TMI-2, Eastern Stories and Elveszett Vilag support it. Probably all derivate of eachother and the original LPMud. Another change (not so important, but quite obvious) is the use of [Pinkfish colour codes](../other/pinkfish); see the list of codes for details.<br/>
A more recent addition is TCP support for Intermud 2 messages. This may be based on the MudOS or Eastern Stories driver which offer native TCP support. See the description of the startup packet in the message list for more details.
:::
## *Introduction*

Intermud uses Game Port +4 as a default

This file gives a description of how CDlibs and CDdrivers can communicate with each other and the protocol used.

Intermud communication is done via udp packages, contrary to players connected to the mud whose communication goes through tcp. This has two consequences:

- Intermud communication will be unreliable. This is the nature of UDP. 
- Only two file descriptors are used for all intermud communication,  not one / mud.

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
- Every message consists of a 'command' and 'parameters'. Paramaters can     have subparameters.

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
This command is sent to the 'mudlist server' when the mud   starts. The 'mudlist server' is defined in /secure/std.h   Typical parameter tags are:      

  * NAME"The name of the mud, for example "Genesis" 
  * "VERSION"The version of the GD, for example "CD.01.57"         
  * "MUDLIB"The version of the mudlib, ex. "CD.00.25" 
  * "HOST"The hostname running the mud
  * "PORT"The portnumber for players to connect to
  * "PORTUDP"The portnumber to receive udp messages on
  * "TIME"The localtime of startup, UNIX ctime-format   

### "shutdown"
This command is sent to the mudlist server when the mud   shuts down. Typical parameters are the same as for startup.

### "ping_q"
This command is sent from one mud to another to request   information. Typical parameters sent are: 

  * "NAME"The name of the mud, for example "Genesis"   
  * "PORTUDP"The portnumber to receive udp messages on   

### "ping_a"
This command is sent as an answer to a 'ping query' command.   The parameters are the same as in the startup command.   

### "mudlist_q"
This command is sent from one mud to another to get info   on what muds it knows about. Parameters are the same as   for a 'ping_q' command.

### "mudlist_a"
This command is sent as an answer to a 'mudlist query'   command. The parameters are simple numeric indexes, one   for each known mud. For each such index is then given   a subparameterlist holding these typical 'subtags':      

  * "NAME	The name of the mud, for example "Genesis"   

  * "HOST        The hostname running the mud   

  * "HOSTADDRESS   The IP adress used to talk with the mud          

  * "PORT"The portnumber for players to connect to   

  * "PORTUDP"The portnumber to receive udp messages on       

  * Typical example:
    ```
    @@@mudlist_a
    ||1:|NAME:GENESIS|HOST:milou
        |HOSTADRESS:129.16.79.12 
        |PORT:2000|PORTUDP:2500
    ||2:|NAME:Othermud|HOST:other
        |HOSTADRESS:127.0.0.1 			
        |PORT:3000|PORTUDP:4500
    @@@
    ```

### warning

This is sent as to warn another mud that someone is trying   to send messages using the same mudname but from another host. 
Typical parameters:
* "MSG"
    A message containing the warning
* "FAKEHOST"
    IP address of the falsifier   

### "gwizmsg"

This is sent from one mud to another holding a message to present on the 'global wizline'. 
Typical parameters are:      
* "NAME"
   Name of the mud where the message is from.   
* "WIZNAME"
   Name of the wizards sending the message   
* "EMOTE"
    The integer 1 if this message is to be interpreted (shown) as an emote (like Koresh smiles), or 0 otherwise. EMOTE is part of the original protocol implementation 
* "GWIZ"
   The actual message text as given by the wizard           

### "gfinger_q"
    Retreive information about a player or creator on a remote mud.
    
    "ASKWIZ"
        The real name of the person who asks the information
    
    "PLAYER"
        The name of the player about who info is asked 

### "gfinger_a"
    The answer to a "gfinger_q"-request: finger information about a player.
    
    "ASKWIZ"
        This value must be the same as in the query
    
    "MSG"
        The available finger information 

### "gtell"
    This was one of the first extensions and is supported by practically every mud which uses this protocol. You use it to tell something to a person on another mud.
    The confirmation of a tell, should come in a "affirmation_a" packet; however many muds simply send a "gtell" message back from a non-existant user (e.g. "root" or "udp-daemon") telling you whether the message was shown to the remote user, or not.
    
    "WIZFROM"
        The name of the person who says something
    
    "WIZTO"
        The name of the person to whom is spoken
    
    "MSG"
        The actual message 

### "affirmation_a"
    This is the affirmation of a "gtell" message.
    
    "TYPE"
        This value always is "gtell"
    
    "WIZFROM"
        The name of the person who received the tell message
    
    "WIZTO"
        The name of the person to send the confirmation to
    
    "MSG"
        The actual affirmation message 

### "gchannel"
    This is sent from one mud to another holding a message to present on yet another 'global wizline'. It is an extension to the default wizline and was probably introduced by the Eastern Stories mudlib (based on TMI-2). The 'es' channel is supposed to be restricted to MUDs with this mudlib; and the 'twiz' (taiwanese wizards) channel to IP-numbers starting with 140. I say "supposed", because they all show up in my logfiles ;-)
    
    "USRNAME"
        Name of the wizard sending the message in ASCII
    
    "CNAME"
        Name of the wizard sending the message in Big5 character set
    
    "CHANNEL"
        The channel this wizards uses: known channels are 'es', 'gwiz', 'twiz' and 'jy'
    
    "MSG"
        The actual message text as given by the wizard (usually in Big5 characters)
    
    "EMOTE"
        The integer 1 if this message is to be interpreted (shown) as an emote (like Koresh smiles), or 0 otherwise 

### "locate_q"
    Check whether a certain player is logged on (or exists) in this mud. This request is usually send to all known muds at once.
    
    "ASKWIZ"
        The real name of the person who asks the information
    
    "TARGET"
        The name of the player we are looking for 

### "locate_a"
    The answer to a locate-request. Some muds only send a reply when the person was actually located (and leave out the "LOCATE" parameter).
    
    "ASKWIZ"
        This value must be the same as in the query
    
    "TARGET"
        This value must be the same as in the query
    
    "LOCATE"
        "yes" or "no", depending on success 

### "mail_q"
    This extension was first added in August 1993 by Inspiral@Tabor. Use this to send an intermud mail message to somebody at another mud. Note that mail messages are usually longer than the maximum size of an UDP packet, so they will be split up in several packets. Some MUDs only support these messages over TCP connections.
    
    "WIZTO"
        The mail recipient
    
    "WIZFROM"
        The mail sender
    
    "DATE"
        A timestamp for the mail (usually in seconds since 1-1-1970)
    
    "SUBJECT"
        A subject for the mail message
    
    "CC"
        Optional adresses for extra recipients
    
    "ENDMSG"
        The value is "1" iff this is the last packet of a message, it is usually omitted for preceding messages This should always be 1 if the message fits in a single packet
    
    "MSG"
        The contents of the mail: this may be a multi-line string 

### "mail_a"
    Confirmation for a received intermud mail message.
    
    "ENDMSG"
        Present with value "1" iff the mail_q had this set as well
    
    "RESEND"
        Optional: ask the originator MUD to resend a message that could not be processed. I've never seen this option actually being used... 

### "rwho_q"
    This extensions is used very often as well. It will list the users on the remote mud.
    Only one parameter is used:
    
    "ASKWIZ"
        The real name of the person who asks the information 

### "rwho_a"
    The reply to a rwho query. There is no standard layout for the reply; it should be shown directly to the player who requested the info. Admins are free to include as little or as much information as they like (titles, levels, etcetera).
    
    "ASKWIZ"
        This value must be the same as in the query
    
    "RWHO"
        The string listing all users on the remote mud 

### "supported_q"
    Check whether a remote mud supports a certain intermud service.
    
    "CMD"
        Any intermud command (e.g. "ping_q")
    
    "ANSWERID"
        An unique id for recognizing the answer (this can be the name of a player when the message is initiated by one). 

### "supported_a"
    In the original protocol, this (undocumented) command returns the "SUPPORTED" parameter with value "yes" when the command is supported and "NOTSUPPORTED" with value "yes" otherwise. However some implementations appear to be using the "no" value for "SUPPORTED" in the latter case. Either way: a mud should not really depend on the answer to this query.
    
    "CMD"
        This value must be the same as in the query
    
    "ANSWERID"
        This value must be the same as in the query
    
    "SUPPORTED"
        The only defined value is "yes"
    
    "NOTSUPPORTED"
        The only defined value is "yes" (To avoid confusion, SUPPORTED and NOTSUPPORTED should never be used in combination!) 


​                
## Future extensions

There are numerous possibilities for new commands in the protocol. Some  examples of what could be added are commands to handle:

- Remote who, what wizards/players are logged in on a specific mud
- Remote finger - Give special information on a wizard/player
- Remote tell - Direct communication to a specific wizard/player
- Remote control - Control an NPC in a foreign mud.
- Intermud mail - Send mail to someone at somemud
- Simple mud to mud filetransfer.

Further in the future are:

- Capabilities to transfer source code between muds.    Perhaps having a special 'foreign mud' domain, where each 'wizard'    is another mud.
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

----
##  Was ist InterMud2

Hier eine sehr kurze Zusammenstellung was das ist, was wir immer InterMud2 nennen. Das genaue Protokoll ist in den entsprechenden 'Standards' nachzulesen, zum Beispiel die Hilfeseiten in der LDmud Distribution (zB hier Basics und Protokoll).

    Kommunikation zwischen zwei Muds (Broadcasts werden durch n Einzelkommunikationen erreicht).
    Verwendet UDP mit 1 kB Paketen.
    Darauf aufsetzend das PKT Protokoll
    Darauf wiederum aufsetzend das eigentliche I2 Protokoll. Bei Paketen < 1 kB darf das zwischengeschaltete PKT Protokoll wegfallen und I2 wird direkt über UDP gesendet.

Historisch gesehen ist das benutzte Protokoll in Wirklichkeit Zebedee Intermud mit Alvin@Sushi's Mail Extension. Beides ist eine Erweiterung des eigentlichen Intermud2 aus dem CD.

Die I2 Pakete sind Strings der Art "key:value|key2:value2|key3:value3" . Da es für ':' und '|' keine Escapes gibt, dürfen diese Zeichen nicht in Keys oder Values vorkommen. Dies ist natürlich für Nutzertexte untragbar. Deswegen werden die Nutzdaten immer in einem Feld mit dem Schlüssel "DATA" übertragen. Er muss immer am Ende des Paketstrings liegen, da in seinem Value keine Trenntoken mehr gesucht werden.

Da das verbindungslose UDP benutzt wird, kann man nicht sicher sein, ob ein Paket das Zielmud erreichte. Um dieses etwas auszugleichen werden bei den meisten Pakettypen Replypakete verwendet. Kommt kein Reply wird die Nachricht nochmals gesendet (meistens bis zu maximal 3 Mal).

Alle Nutzdaten basieren auf Texten, die direkt an den Spieler ausgegeben werden. Insbesondere kann eine Mudlib schwer unterscheiden ob ein Reply-Text eine Fehlermeldung oder ein OK war.

Broadcasts wie Channelmeldungen werden durch einzelnes Ansprechen aller 'bekannten' Muds erreicht, also für jedes Mud ein Paket erzeugt und gesendet. Oftmals gehen hierbei aber Pakete verloren, da Netzrouter auf dem Weg bei einem Massenansturm von > 100 UDP Paketen einfach welche verwerfen (dürfen). Ausserdem muss in jedem Mud die Mudliste aktuell gehalten werden, was sich in der Praxis als mehr oder weniger schwierig erweist (auch durch weitere Protokollmängel). 

## MUDs using Intermud2

| MUD                  | IP                                        | Intermud 2 Port |
| -------------------- | ----------------------------------------- | --------------- |
|                      |                                           |                 |
| Universes            | [108.252.255.105](http://108.252.255.105) | 3341            |
| OuterSpace           | [159.69.87.242](http://159.69.87.242)     | 3001            |
| jy_record            | [1.34.90.150](http://1.34.90.150)         | 6670            |
| BrutaliaII           | [79.120.193.52](http://79.120.193.52)     | 4452            |
| The Dream Of Seven   | [210.59.236.38](http://210.59.236.38)     | 7004            |
| Time Space Dreamland | [210.59.236.38](http://210.59.236.38)     | 5505            |
| Fantasy Space        | [210.59.236.38](http://210.59.236.38)     | 5559            |
| Revival World        | [210.59.236.38](http://210.59.236.38)     | 4004            |
| The Admin Mud III    | [210.59.236.38](http://210.59.236.38)     | 3004            |
| MYSTICISM-MUD        | [124.223.67.13](http://124.223.67.13)     | 2027            |
| TimMUD               | [104.154.76.197](http://104.154.76.197)   | 5563            |
| BrutalDev            | [79.120.193.52](http://79.120.193.52)     | 4456            |
| CsillagKod           | [91.205.173.162](http://91.205.173.162)   | 7785            |
| ElveszettVilag2      | [78.131.8.19](http://78.131.8.19)         | 3341            |
| ElveszettVilag       | [78.131.8.19](http://78.131.8.19)         | 6674            |
| Limbo                | [91.205.173.162](http://91.205.173.162)   | 9008            |
| Karhozat             | [91.205.173.162](http://91.205.173.162)   | 4463            |
| Aranylaz             | [91.205.173.162](http://91.205.173.162)   | 4329            |
| Bastard              | [91.205.173.162](http://91.205.173.162)   | 2230            |
| Illusory of time     | [220.133.199.45](http://220.133.199.45)   | 7004            |
| DarkeMUD             | [50.38.11.140](http://50.38.11.140)       | 5567            |