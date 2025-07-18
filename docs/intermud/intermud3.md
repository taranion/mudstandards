---
sidebar_label: Intermud3
---

# InterMUD 3

## Architecture
Intermud3 (*I3*) is an architecture that relies on a (or more) central server, called "Router". MUD server connect to a hub 
and relay all messages through it. This makes the router server a single point of failure for the network.<br/>
To counter that it is possible to have more than one hub. I3 handles [*Inter-Router* communication](https://wotf.org/i3/irn/v2/).
In that case a MUD server connects to any router and during the setup and 
stays there, so that every router maintains his own MUD list. Routers synchronize
their MUD lists and pass on messages targeted for servers connected to a peer router.

```mermaid
architecture-beta
    group api(cloud)[Central Hubs]

    service h1(database)[Hub 1] in api
    service h2(database)[Hub 2] in api
    service m1(server)[MUD 1]
    service m2(server)[MUD 2]
    service m3(server)[MUD 3]
    service m4(server)[MUD 4]
    service m5(server)[MUD 5]

    h1:L -- R:h2
    m3:R -- L:h2
    m4:R -- L:h2
    m1:L -- T:h1
    m2:L -- R:h1
    m5:L -- B:h1
```
### Central router
Information on the central router can be found on [http://lpmuds.net/intermud.html](http://lpmuds.net/intermud.html). 
As of now (July 2025) there are the following routers documented:

| Name       |   IP Address    | Port  | Online     | Remark |
| ---------- | --------------- | ----- | --------- | ------ |
| *dalet     |  97.107.133.86  | 8787  | Yes       |        |
| *i4        | 204.209.44.3    | 8080  | Yes       |        |
| *Kelly     | 150.101.219.57  | 8080  | -         |        |
| *wpr       | 136.144.155.250 | 8080  | -         |        | 
| *wir       | 136.144.155.250 | 3004  | -         | Server to test implementations against |


## Message Encoding and Transport
Due to its origin in LPC muds, *I3* definitions are based on LPC datastructures 
and their encoding named [**mudmode**](https://wotf.org/specs/mudmode.html).
Basically a *mudmode* encoded message is a null-terminated string of undefined 
encoding, that is preceeded by a 4 byte header that contains the length of the 
message string (including terminating 0 byte). 
```mermaid
---
title: "MUDMode encoding"
---
packet
0-3: "Length"
4-20: "Message String"
21: "\\0"
```
Messages are sent via TCP to the central router. The connection is kept open - 
the server sends answers or requests on the same connection.

The MUDMode encoded message strings look a bit like JSON, but aren't related. Here is one example:
```
({"mudlist",5,"*dalet",0,"GraphicMUD",0,550962,(["Dead_Souls_kalle":0,"DikuMUD":0,"Pull-A-Party":0,"Unnamed_CoffeeMUD#2020103645":0,"CoffeeMud perso":0,"Codes Mind":0,"Your Muds Name131891405":0,"XanMUD":0,"Fabled Lands":0,"Antikaria":({0,"128.16.10.197",9000,0,0,"antikaria","TMD.BaseLib","TMD","TMD","mudlib development","cs1tjc@cs.bath.ac.uk",(["finger":1,"who":1,"channel":1,"locate":1,"ucache":1,"emoteto":1,"tell":1,]),(["Written in":"c#",]),}),]),})
```
Formatted for easier reading (btw, this is a [mudlist](#mudlist) event)
```
({
    "mudlist",
    5,
    "*dalet",
    0,
    "GraphicMUD",
    0,
    550962,
    ([
        "Dead_Souls_kalle":0,
        "DikuMUD":0,
        "Pull-A-Party":0,
        "Unnamed_CoffeeMUD#2020103645":0,
        "CoffeeMud perso":0,
        "Codes Mind":0,
        "Your Muds Name131891405":0,
        "XanMUD":0,
        "Fabled Lands":0,
        "Antikaria":({
            0,
            "128.16.10.197",
            9000,
            0,
            0,
            "antikaria",
            "TMD.BaseLib",
            "TMD",
            "TMD",
            "mudlib development",
            "cs1tjc@cs.bath.ac.uk",
            (["finger":1,"who":1,"channel":1,"locate":1,"ucache":1,"emoteto":1,"tell":1,]),
            (["Written in":"c#",]),
        }),
    ]),
})
```
:::warning[Warning for non LPC developer]
Beware that arrays and maps - if they contain at least one element - are comma-separated and have a closing comma, even if there isn't any data following.
It seems to be syntactically invalid sending data without that final comma.

MUDMode does not allow whitespaces between the elements.

The integer ``0`` seems to operate as a kind of null pointer and can be used even there where strings or other types are expected.
:::

```
--> ({"startup-req-3",5,"GraphicMUD",0,"*dalet",0,-1,0,0,4000,6073,0,"No LPC","No LPC","None","GraphicMUD","mudlib development","spamprotect@example.com",(["who":1,]),([]),})
<-- ({"startup-reply",5,"*dalet",0,"GraphicMUD",0,({({"*dalet","97.107.133.86 8787",}),}),661222234,})
<-- ({"mudlist",5,"*dalet",0,"GraphicMUD",0,550962,(["Dead_Souls_kalle":0,"DikuMUD":0,"Pull-A-Party":0,"Unnamed_CoffeeMUD#2020103645":0,"CoffeeMud perso":0,"Codes Mind":0,"Your Muds Name131891405":0,"XanMUD":0,"Fabled Lands":0,"Antikaria":({0,"128.16.10.197",9000,0,0,"antikaria","TMD.BaseLib","TMD","TMD","mudlib development","cs1tjc@cs.bath.ac.uk",(["finger":1,"who":1,"channel":1,"locate":1,"ucache":1,"emoteto":1,"tell":1,]),(["Written in":"c#",]),}),]),})
```

```mermaid
sequenceDiagram
    participant Client
    participant Router
    Client->>Router: ({"startup-req-3",5, ...})
    Note over Client, Router: Client identifies itself to the network
    Router->>Client: ({"startup-reply",5,"*dalet",0, ...})
    Note over Router, Client: Router assigns client a unique ID (password)
    alt Number of packages varies
    Router->>Client: ({"mudlist",5,"*dalet",0,"GraphicMUD",0,...})
    Router->>Client: ({"mudlist",5,"*dalet",0,"GraphicMUD",0,...})
    end
    Note over Router, Client: Router sends all changes in MUD list since last state
```
------
Now follows the copy of the full [Intermud3  spec](https://web.archive.org/web/20070813072129/http://www.intermud.org/i3/protocol.php3)

:::note
**Intermud 3 v1**: [http://www.intermud.org/i3/protocol.php3](https://web.archive.org/web/20070813072129/http://www.intermud.org/i3/protocol.php3)<br/>
**Intermud 3 v3**: [https://wotf.org/specs/i3.html](https://wotf.org/specs/i3.html)

Other sources:
- [Mudmode](https://wotf.org/specs/mudmode.html)
- [Intermud 3 IRN v1](https://wotf.org/i3/irn/v1/)
- [Intermud 3 IRN v1](https://wotf.org/i3/irn/v2/)
:::
Initial protocol design by: Greg Stein John Viega Tim Hollebeek


This document details a proposal for a future generation of Intermud protocols.  It is designed to use the high level communication facilities provided by the MudOS LP driver. Other drivers may be capable of handling the communication protocol, but this proposal does not focus on them. See [Other Drivers](https://web.archive.org/web/20070813072129/http://www.intermud.org/i3/protocol.php3#other) for a possible scheme to include them into the Intermud as defined by this proposal.

This document has the following sections:

- [Contributors](https://web.archive.org/web/20070813072129/http://www.intermud.org/i3/protocol.php3#imps) :        lists the contributors to this specification
- [Logical Network Layout](https://web.archive.org/web/20070813072129/http://www.intermud.org/i3/protocol.php3#network) :        describes how the muds connect to each other
- [Packet Format](https://web.archive.org/web/20070813072129/http://www.intermud.org/i3/protocol.php3#formats) :        describes the basic format of all packets
- [Services](https://web.archive.org/web/20070813072129/http://www.intermud.org/i3/protocol.php3#services) :        describes the services available
- [Support Packets](https://web.archive.org/web/20070813072129/http://www.intermud.org/i3/protocol.php3#support) :        describes additional packet types for maintainance purposes
- [OOB Protocols](https://web.archive.org/web/20070813072129/http://www.intermud.org/i3/protocol.php3#oob) :        describes the protocols used for OOB communications
- [Router Design](https://web.archive.org/web/20070813072129/http://www.intermud.org/i3/protocol.php3#router) :        describes the design of the routers
- [Other Drivers/Mudlibs](https://web.archive.org/web/20070813072129/http://www.intermud.org/i3/protocol.php3#other) :        describes an approach that could be used to include        non-MudOS drivers
- [Error Summary](https://web.archive.org/web/20070813072129/http://www.intermud.org/i3/protocol.php3#errors) :        quick summary of the error codes used
- [Packet Types Summary](https://web.archive.org/web/20070813072129/http://www.intermud.org/i3/protocol.php3#types) :        quick summary of the packets that are used
- [Compressed Mode](https://web.archive.org/web/20070813072129/http://www.intermud.org/i3/protocol.php3#compressed) :        describes a scheme for compressing the transmitted data
- [Change Log](https://web.archive.org/web/20070813072129/http://www.intermud.org/i3/protocol.php3#changelog) :        lists recent changes to the specification

------

## Contributors

Many people have contributed to this specification.  This is an attempt to list those people who have helped in some way or another.

Contributors:

-  As mentioned at the head of this page, the core designers of this protocol were:

  - Deathblade (Greg Stein)
  - Rust (John Viega)
  - Beek (Tim Hollebeek)

  

-  Descartes (George Reese)

  Descartes contributed in a number of areas, particularly as one of the pioneer implementors after the initial development by the Lima Mudlib team. 

  

-  Deathknight (Jesse McClusky)

  Deathknight was the original contributor of the central router-based, backbone design of the current I3 system. 

  

There are, of course, many other contributors who offered input both at the conference in February '95 and on the intermud mailing list. Their omission is not by design, but because informartion wasn't available at the time of this writing (they didn't provide info). 



------

## Logical Network Layout

The logical network of muds is organized into a set of fully connected routers each acting as a hub for an arbitrary set of muds.

A mud has a "preferred" router, but will be told and will record information about all routers in existence (for failover in case the preferred is down).  A mud's preferred router may be changed programatically to handle load balancing among routers.

The routers open and maintain TCP sessions (using MudOS's "MUD" mode) to each of the other routers (fully connected network).  Each router will hold a list of all muds within the intermud and which router the mud is connected to.

An active TCP session (in MUD mode) is maintained between a mud and its router. The mud is responsibile for opening (see the [startup-req-2](https://web.archive.org/web/20070813072129/http://www.intermud.org/i3/protocol.php3#startup-req-2) packet), maintaining, reconnecting, and shutting down this session.  A graceful exit will cause the router to propogate information about the "down" state of the mud to the other routers and muds on the intermud.  A dropped connection will wait 5 minutes before delivering "down" state notification (this 5 minutes is provided for the mud to reconnect).  A mud is removed after 7 days of being in the down state.

The network of TCP sessions between the routers and muds will be used for "in band" transmissions.  The data carried over these connections will be limited to "fast response" messages.  Some services will be carried off of this network and are called "out of band" (OOB) transmissions.

Each mud will listen at a TCP port for incoming connections for processing OOB transmissions.  At the moment, the services that use the OOB transmissions are mail, news, and file transfers.  Connections will be opened as needed and closed when the transmission completes.

Lastly, each mud may maintain a UDP port for some specific OOB transmissions.  At the moment, though, there are no UDP-based OOB services, so this port is typically not opened.

### MUD Naming

For proper identification within the Intermud network, muds must use a canonical naming system.  The canonical name is defined to be the mud's actual name (properly capitalized, with spaces, etc). Examples: *Quendor*, *Idea Exchange*.  These names will be used in the mudlist and in all packet routing.  The case is significant, although the routers may (?) disallow two muds with the same name, but with different casing.

Routers will have names assigned to them; there are a few cases where a router must be referenced.  routers will be distinguished from muds with a leading asterisk (such as **nightmare*). Muds may not define a name with a leading asterisk if they wish to be a part of the Intermud.



------

## Packet Format

Transmissions are LPC arrays with a predefined set of six initial elements: `({ type, ttl, originator mudname, originator username, target mudname, target username, ... })`.

type describes the type of the packet. See [Packet Types](https://web.archive.org/web/20070813072129/http://www.intermud.org/i3/protocol.php3#types) for a summary of the packet types used in this protocol.

ttl is the packet's Time To Live (TTL). This is similar to an IP packet's TTL - it specifies the number of hops left to a packet.  This may not be absolutely necessary, but provides a mechanism to handle the case where the routers mis-route a packet into an endless loop.  Eventually, it will time out and be removed from the network.

originator mudname is used to indicate the mud where the packet originated.  If the packet cannot be delivered for some reason, this mud will be notified, if possible.

originator username is used to indicate the user that triggered the delivery of the packet. If the mud itself sent the packet (e.g. mail delivery or shutdown notification), then the username will be 0.  The username should be in lower-case.

target mudname is used to route the packet to the appropriate destination mud. Zero is used to indicate broadcast packets.

target username is used to route the packet to a particular user on a remote mud.  The username may be 0 if the packet is targeted for the mud itself rather than a specific user.  This should always be in lower-case.  The target mud will attempt to find the user with whatever appropriate means.

Many packets will specify a visname.  This is a user's "visible" name - the name which should be displayed to other users. Typically, this is equivalent to the username with altered capitalization, but may actually be quite arbitrary with respect to the username.



------

## Services

There are eleven services covered by this proposal:

- [tell](https://web.archive.org/web/20070813072129/http://www.intermud.org/i3/protocol.php3#tell) :        send a message to a user on a remote mud
- [emoteto](https://web.archive.org/web/20070813072129/http://www.intermud.org/i3/protocol.php3#emoteto) :        send an emote to a user on a remote mud
- [who](https://web.archive.org/web/20070813072129/http://www.intermud.org/i3/protocol.php3#who) :        get a list of users on a remote mud
- [finger](https://web.archive.org/web/20070813072129/http://www.intermud.org/i3/protocol.php3#finger) :        get information about a particular user on a remote mud
- [locate](https://web.archive.org/web/20070813072129/http://www.intermud.org/i3/protocol.php3#locate) :        locate a player on a remote mud(s)
- [channel](https://web.archive.org/web/20070813072129/http://www.intermud.org/i3/protocol.php3#channel) :        send a string/emote/soul between muds
- [news](https://web.archive.org/web/20070813072129/http://www.intermud.org/i3/protocol.php3#news) :        propogate news posts between muds
- [mail](https://web.archive.org/web/20070813072129/http://www.intermud.org/i3/protocol.php3#mail) :        propogate mail items between muds
- [file](https://web.archive.org/web/20070813072129/http://www.intermud.org/i3/protocol.php3#file) :        transfer a file between muds
- [auth](https://web.archive.org/web/20070813072129/http://www.intermud.org/i3/protocol.php3#auth) :        perform mud or user authentication
- [ucache](https://web.archive.org/web/20070813072129/http://www.intermud.org/i3/protocol.php3#ucache) :        cache information about remote users

### Service: tell

The originator will deliver the following packet to the target mud over the in-band network (to its router):

```
    ({
        (string) "tell",
        (int)     5,
        (string) originator_mudname,
        (string) originator_username,
        (string) target_mudname,
        (string) target_username,
        (string) orig_visname,
        (string) message
    })
```

At the target mud, the message will be delivered to target_username. The orig_visname is told to the target_username to indicate who sent the message; it is usually combined with the originator_mudname as: `sprintf("%s@%s", orig_visname, originator_mudname)`.

The target_username should be lower-cased by the originating mud.

If the router fails to deliver the packet for some reason, it will return an [error](https://web.archive.org/web/20070813072129/http://www.intermud.org/i3/protocol.php3#error) packet.

### Service: emoteto

The originator will deliver the following packet to the target mud over the in-band network (to its router):

```
    ({
        (string) "emoteto",
        (int)    5,
        (string) originator_mudname,
        (string) originator_username,
        (string) target_mudname,
        (string) target_username,
        (string) orig_visname,
        (string) message
    })
```

At the target mud, the message will be delivered to target_username with the appropriate formatting.  The message will contain $N tokens for substituting the originator's name.  This name is typically formatted using: `sprintf("%s@%s", orig_visname, originator_mudname)`.

The target_username should be lower-cased by the originating mud.

A simple example would be a message such as, "$N smiles at you." which would be transformed into something like, "Joe@PutzMud smiles at you."

If the router or target mud fails to deliver the packet for some reason, it will return an [error](https://web.archive.org/web/20070813072129/http://www.intermud.org/i3/protocol.php3#error) packet.

### Service: who

The originator will deliver the following packet over the in-band network (to its router):

```
    ({
        (string) "who-req",
        (int)    5,
        (string) originator_mudname,
        (string) originator_username,
        (string) target_mudname,
        (string) 0
    })
```

The router will route the packet to another router or to the target mud.  The target mud returns:

```
    ({
        (string)  "who-reply",
        (int)     5,
        (string)  originator_mudname,
        (string)  0,
        (string)  target_mudname,
        (string)  target_username,
        (mixed *) who_data
    })
```

where who_data is an array containing an array of the following format for each user on the mud:

```
    ({
        (string)  user_visname,
        (int)     idle_time,
        (string)  xtra_info
    })
```

Each user_visname should specify the user's visual name. idle_time should be measured in seconds and xtra_info should be a string.

If the router fails to deliver the packet for some reason, it will return an [error](https://web.archive.org/web/20070813072129/http://www.intermud.org/i3/protocol.php3#error) packet.

### Service: finger

Finger operates similarly to [who](https://web.archive.org/web/20070813072129/http://www.intermud.org/i3/protocol.php3#who) but returns information about a specific user rather than all users logged into the mud.  This service uses the in-band network; the packet has the following format:

```
    ({
        (string) "finger-req",
        (int)    5,
        (string) originator_mudname,
        (string) originator_username,
        (string) target_mudname,
        (string) 0,
        (string) username
    })
```

Note: technically, we could use the target_username field to hold the name of the person to finger, but we do not wish to imply that the packet is *destined* for that user. The packet is only querying information about the user.

The target mud will return:

```
    ({
        (string) "finger-reply",
        (int)    5,
        (string) originator_mudname,
        (string) 0,
        (string) target_mudname,
        (string) target_username,
        (string) visname,
        (string) title,
        (string) real_name,
        (string) e_mail,
        (string) loginout_time,
        (int)    idle_time,
        (string) ip_name,
        (string) level,
        (string) extra  // eg, a .plan file, or any other relevant info
    })
```

A mud may return 0 for any item if they wish to keep the information private.  In particular, it is suggested that information about players (as opposed to wizards) be kept confidential.

The returned visname should contain the user's visual name. loginout_time specifies the (local) time the user logged in (if they are currently on) or the time the user logged out.  The value should be expressed as a string.  It should be 0 to indicate no information.  The idle_time is expressed as an integer number of seconds of idle time.  If this value is -1, then the user is not logged onto the mud at the moment.

If extra is given, then it should be terminated with a carriage return.



### Service: locate

This service locates a particular user on the Intermud system. The following packet is delivered over the in-band network (to the mud's router), which then delivers it to all muds (target_mudname == 0):

```
    ({
        (string) "locate-req",
        (int)    5,
        (string) originator_mudname,
        (string) originator_username,
        (string) 0,
        (string) 0,
        (string) username
    })
```

**If** the requested user is logged into the receiving mud, then the following reply is returned:

```
    ({
        (string) "locate-reply",
        (int)    5,
        (string) originator_mudname,
        (string) 0,
        (string) target_mudname,
        (string) target_username,
        (string) located_mudname,
        (string) located_visname,
        (int)    idle_time,
        (string) status,
    })
```

located_visname should contain the correct visual name of the user. idle_time will have the idle time (in seconds) of the located user. 

status specifies any special status of the user.  This will typically be zero to indicate that the user has no special status. The values for this string are arbitrary, but certain statuses have predefined values that can be used:

-  "link-dead"
-  "editing"
-  "inactive"
-  "invisible"
-  "hidden"

These predefined values are intended to cover those statuses that might modify a user's reception of Intermud transmissions; they are not intended to be all-inclusive (and the string is arbitrary in any case). The predefined way to specify multiple attributes is with a comma-space-separated list such as: "editing, hidden".  The located mud should not attempt to apply special formatting or other characters, instead leaving that to the receiving mud. 

Other examples for the status string might be "afk for dinner" or "taking a test". 



### Service: channel

There are two types of channels in the Intermud system: selective admission and selective banning.  All channels are owned by a particular mud and are adminstrated by that mud only.  The owner of a channel may also choose to filter the channel, although this may subject that mud to an increased load for processing the channel contents and the channel will become unavailable when the host mud is down.

The routers maintain three channel lists - one list for each type of channel (selective admission vs banning) where the channel is unfiltered, and one list for selective admission, filtered channels.  Selectively banned, filtered channels are not allowed. For each channel, the router will store which type it is (what list it is on), the owning mud of the channel, and a list of muds that are admitted/banned.

When a mud sends a startup-req-2 packet, it includes its chanlist-id in the packet.  The router will potentially respond with a chanlist-reply message to update the mud's channel list.

The router will respond to channel list changes with the chanlist-reply packet.

```
    ({
        (string)  "chanlist-reply",
        (int)     5,
        (string)  originator_mudname,     // the router
        (string)  0,
        (string)  target_mudname,
        (string)  0,
        (int)     chanlist_id,
        (mapping) channel_list
    })
```

channel_list is mapping with channel names as keys, and an array of two elements as the values. If the value is 0, then the channel has been deleted. The array contains the host mud, and the type of the channel: `        0  selectively banned        1  selectively admitted        2  filtered (selectively admitted) ` All channel messages are delivered to the router.  It will then pass the message to the appropriate set of muds.  If the channel is filtered, then the packet will be delivered to the host mud for filtering; it will then return to the router network for distribution.  It is assumed that a channel packet for a filtered channel that comes from the channel host has been filtered.

Channel messages come in three flavors: standard messages, emotes, and targetted emotes. These use packets channel-m, channel-e, and channel-t, respectively. They are:

```
    ({
        (string) "channel-m",
        (int)    5,
        (string) originator_mudname,
        (string) originator_username,
        (string) 0,
        (string) 0,
        (string) channel_name,
        (string) visname,
        (string) message
    })
    ({
        (string) "channel-e",
        (int)    5,
        (string) originator_mudname,
        (string) originator_username,
        (string) 0,
        (string) 0,
        (string) channel_name,
        (string) visname,
        (string) message
    })
    ({
        (string) "channel-t",
        (int)    5,
        (string) originator_mudname,
        (string) originator_username,
        (string) 0,
        (string) 0,
        (string) channel_name,
        (string) targetted_mudname,
        (string) targetted_username,
        (string) message_others,
        (string) message_target,
        (string) originator_visname,
        (string) target_visname
    })
```

When a mud receives a channel-m packet, it should deliver it locally to all listeners.  The actual message delivered to users should be a composition of the originator_mudname, visname, channel_name, and message.  The message should not be preformatted with this information before delivery so that individual muds can define the display semantics. A suggested format is:

```
[gwiz] John@Doe Mud: help me! help me! I am a cluebie newless!
```

This was composed with:

```
 sprintf("[%s] %s@%s: %s", channel_name, visname, originator_mudname, message); 
```

channel-e packets are similar to channel-m packets, but the message has a token in it to represent where the originator's name should be. This token is $N.  An example is "$N smiles."

\### need more info here ###

When a mud receives a channel-t packet, it should deliver the message locally to all listeners.  The actual message delivered should be a composition of the channel_name and one of message_others or message_target.  The messages will include $N and $O for the name of the originator and the object/target of their emote. The appropriate message is selected based on the listener - if the listener matches the targetted mud/user, then the message_target should be used. A suggested format is:

```
[gwiz] With a flying leap, John@Doe Mud falls into the pit.
[gwiz] Jane@BlandMud waves to Goober@PutzMud.
[gwiz] Jane@BlandMud waves to you.
```

These were composed with:

```
 sprintf("[%s] %s", channel_name, one_of_the_messages); 
```

\### need a bit more on channel-t ###

All messages should *not* be terminated with a newline - that will be applied on the receiving mud if necessary. The target_username should be in lower-case if provided.

A channel may be added with the following packet:

```
    ({
        (string) "channel-add",
        (int)    5,
        (string) originator_mudname,
        (string) originator_username,
        (string) target_mudname,         // the router
        (string) 0,
        (string) channel_name,
        (int)    channel_type
    })
```

The for channel_type field accepts the [same values](https://web.archive.org/web/20070813072129/http://www.intermud.org/i3/protocol.php3#channel-type-list) that the name server returns for channels. Sending this packet again with a new value for channel_type will change the information.

A channel may be removed by the mud that created it by sending the following packet:

```
    ({
        (string) "channel-remove",
        (int)    5,
        (string) originator_mudname,
        (string) originator_username,
        (string) target_mudname,         // the router
        (string) 0,
        (string) channel_name
    })
```

To administer a channel, the channels host/owner mud may use the following packet:

```
    ({
        (string)   "channel-admin",
        (int)      5,
        (string)   originator_mudname,
        (string)   originator_username,
        (string)   target_mudname,         // the router
        (string)   0,
        (string)   channel_name,
        (string *) add_to_list,
        (string *) remove_from_list
    })
```

The muds specified in add_to_list are added to the allowed/banned list stored within the router network.  The muds specified in remove_from_list are removed from that list.

To filter a channel, the following packets will be used :

```
    ({
        (string)   "chan-filter-req",
        (int)      5,
        (string)   originator_mudname,     // the router
        (string)   0,
        (string)   target_mudname,         // the owner/host mud
        (string)   0,
        (string)   channel_name,
        (mixed *)  packet_to_filter,
    })
```

Where packet_to_filter is a channel-m, channel-e or channel-t packet.

The filtered packet is returned to the router in the following packet :

```
    ({
        (string)   "chan-filter-reply",
        (int)      5,
        (string)   originator_mudname,    // The channel host/owner mudname
        (string)   0,
        (string)   target_mudname,        // the router
        (string)   0,
        (string)   channel_name,
        (mixed *)  filtered_packet,
    })
```

A list of who is listening to a channel on a remote mud may be requested with the following packet:

```
    ({
        (string) "chan-who-req",
        (int)    5,
        (string) originator_mudname,
        (string) originator_username,
        (string) target_mudname,
        (string) 0,
        (string) channel_name
    })
```

The reply for the who request takes the following format:

```
    ({
        (string)   "chan-who-reply",
        (int)      5,
        (string)   originator_mudname,
        (string)   0,
        (string)   target_mudname,
        (string)   target_username,
        (string)   channel_name,
        (string *) user_list
    })
```

The user_list should be an array of strings, representing the users' "visual" names.

​         A mud may decide whether or not it is listening to any given channel by sending a channel-listen packet.  This packet is also used to tune out a channel, which should be done whenever no one on the mud is listening to the channel.  The format of this packet is:

```
    ({
        (string) "channel-listen",
        (int)    5,
        (string) originator_mudname,
        (string) 0,
        (string) target_mudname,         // the router
        (string) 0,
        (string) channel_name,
        (int)    on_or_off
    })
```

The on_or_off will contain one of the following values:

```
        0 The mud does not wish to receive this channel.
        1 The mud wishes to receive this channel.
```

Lastly, for targetted emotes, it is necessary to get information on the target.  There are two pieces of information required: the visname and the gender of the target.  This operation is performed with the following packets:

```
    ({
        (string) "chan-user-req",
        (int)    5,
        (string) originator_mudname,
        (string) 0,
        (string) target_mudname,
        (string) 0,
        (string) username
    })
```

Note that the username is separate since the packet is not targetted *to* the user.

The reply for the user info request takes the following format:

```
    ({
        (string) "chan-user-reply",
        (int)    5,
        (string) originator_mudname,
        (string) 0,
        (string) target_mudname,
        (string) 0,
        (string) username,
        (string) visname,
        (int)    gender
    })
```

The gender takes one of the following values: `        0  male        1  female        2  neuter ` Note that a mud may have more variants on gender, but most human languages only have three forms at most.  These are used to select the appropriate pronouns, possessives, and reflexive words.



### Service: news

\### work to do here... this is an OOB service, also employing the auth service ### 

This packet is used to transmit a request for a post to a remote mud's news server (via the OOB network). This connection should not disconnect from the server until it is done dealing with it for the forseeable future.  Instead, it should exchange messages in lockstep with the server.

```
    ({
        (string) "news-read-req",
        (int)    5,
        (string) originator_mudname,
        (string) originator_username,
        (string) 0,
        (string) 0,
        (string) newsgroup_name,
        (int)    id,
    })
```

The server response for this request takes the following form:

```
    ({
        (int)    posting_time,
        (string) thread_id,
        (string) subject,
        (string) poster,
        (string) contents
    })
```

To post a message, send the following packet:

```
    ({
        (string) "news-post-req",
        (string) originator_mudname,
        (int)    mud_login_port,
        (string) mewsgroup,
        (string) thread_id,
        (string) subject,
        (string) poster,
        (string) contents
    })
```

Notice that this packed does not follow the standard packet form, because it is not transmitted over the in band network. The server response should be an integer representing the id of the post, or an error packet.

To request a list of newsgroups, send the following packet:

```
    ({
        (string) "news-grplist-req"
    })
```

The response from the server should be an array containing an array for each available group.  This array should contain the following data:

```
    ({
        (string) group_name,
        (int)    first_post_id,
        (int)    last_post_id
    })
```

### Service: mail

This packet is used to deliver a mail message to a remote mud. It will be delivered over the OOB network (i.e., over a direct TCP connection  to the target mud's out-of-band TCP port).  It has the following form:

```
    ({
        (string)   "mail",
	(int)      id,
	(string)   orig_user,
	(mapping)  to_list,
        (mapping)  cc_list,
        (string *) bcc_list,
        (int)      send_time,
        (string)   subject,
        (string)   contents,
    })
```

Where  to_list  and  cc_list  are mappings of the following format : 

```
    ([
        MUD-A : ({ user-1, user-2, }),
        MUD-B : ({ user-1, user-2, }),
    ])
```

and  bcc_list  is an array of the users at the target mud only.

Mail is acknoleged by the use of a "mail-ack" packet.

```
    ({
        (string)   "mail-ack",
        (mapping)  ack_list,
    })
```

Where ack_list is a mapping whose keys are the id's that have been acknowleged, and the values are arrays of the failures.  NB: **mail** is an OOB service, and employs the auth service. Errors must be sent via the OOB network

### Service: file

\### work to do here... this is an OOB service, also employing the auth service ### 

\### this service still in development ### 

This service is used to transfer files between muds.  Before files may be transferred, a session token must be returned from the remote mud using the login protocols.

It will use the out-of-band TCP port and has the following form: 
 To list the files on a remote mud, send the following packet:

```
    ({
        (string) "file-list-req",
        (string) originator_mudname,
        (string) originator_username,
        (string) dir,
    }) 
```

Where dir is the directory to be listed. dir is considered to be relative to a world read/write directory.   Any leading '/' is ignored. The response from the remote mud should be:

```
    ({
        (string)  "file-list-reply",
        (string)  target_username,
        (mixed *) dir_list,
    }) 
```

Where dir_list is an array containing an array of the following format for each file in the reply list:

```
    ({
        (string) fname,
        (int)    fsize,
        (int)    ftime,
    }) 
```

To send a file to a remote mud, send the following packet:

```
    ({
        (string) "file-put",
        (int)    id,
        (string) originator_mudname,
        (string) originator_username,
        (string) remote_fname,
        (string) contents,
    }) 
```

Puts are acknoleged by the following packet :

```
    ({
        (string) "file-put-ack",
        (int)    id,
        (int)    success,
    })
```

To retrive a file from a remote mud, send the following packet:

```
    ({
        (string) "file-get-req",
        (int)    id,
        (string) originator_mudname,
        (string) originator_username,
        (string) remote_fname,
    }) 
```

The remote mud should respond with

```
    ({
        (string) "file-get-reply",
	(int)    id,
        (int)    success,
        (string) contents,
    })
```

Where  success corresponds to the following.

- ​     -3  :  Request Failed (write permission)

  ​     -2  :  Request Failed (read  permission)

  ​     -1  :  Request Failed (fpath error)

  ​      0  :  Request Failed (unknown error)

  ​      1  :  Request successful

The file should only be transfered from a world writeable directory on one mud to a world writable directory on another.
 remote_fname  is relative to the same directory as file-list-* and any leading '/' is ignored.

The size of the file capable of being sent in this manner is limited by the size of the maximum string length on both the sending and receiving muds.

An error response should be returned over the OOB network.



### Service: auth

The *auth* service is used for performing authentication of a mud.  This authentication is used for OOB communications since in-band communication is always authenticated at a mud-level. 

The authentication is used to verify that an incoming OOB connection request is actually from the mud that the request says it is from.  Before the OOB connection is made, the originator sends the following packet (over the in-band network) to the target mud:

```
    ({
        (string) "auth-mud-req",
        (int)    5,
        (string) originator_mudname,
        (string) 0,
        (string) target_mudname,
        (string) 0
    })
```

The target mud generates a unique integer key, associates that with the originating mud with the key, and then returns the key (over the in-band network) to the originator with:

```
    ({
        (string) "auth-mud-reply",
        (int)    5,
        (string) originator_mudname,
        (string) 0,
        (string) target_mudname,
        (string) 0,
        (int)    session_key
    })
```

At this point, the originator may contact the target mud through the OOB port, using the session_key to authenticate that it actually is the purported originator. 

It is possible, and should be accounted for, that a particular originator might issue multiple auth-mud-req packets before establishing an OOB session to the target mud.  Only the *last* request and session_key need to be remembered.  The keys from prior requests may be discarded and connection attempts using them may be rejected. 

Session keys must remain valid for *10 minutes* from receipt of the auth-mud-req packet.  After that point, the target mud may discard the key and reject connection attempts with that key.  After a successful OOB connection is made, the target mud may discard the key (the key should be interpreted as a one-time key). 



### Service: ucache

The *ucache* service is used for maintaining user information caches within the Intermud network.  A mud may cache information that it receives from [chan-user-req](https://web.archive.org/web/20070813072129/http://www.intermud.org/i3/protocol.php3#chan-user-req) packet. To keep this up to date, muds should issue *ucache-update* packets, which are then used by muds that implement the *ucache* service. 

Whenever the contents of a [chan-user-reply](https://web.archive.org/web/20070813072129/http://www.intermud.org/i3/protocol.php3#chan-user-reply) packet would change (visname or gender), then the mud should broadcast the following packet:

```
    ({
        (string) "ucache-update",
        (int)    5,
        (string) originator_mudname,
        (string) 0,
        (string) 0,
        (string) 0,
        (string) username,
        (string) visname,
        (int)    gender
    })
```

The contents of this packet are similar to the [chan-user-reply](https://web.archive.org/web/20070813072129/http://www.intermud.org/i3/protocol.php3#chan-user-reply) packet. 

Note that the router will filter this packet's delivery to only those muds that support the *ucache* service.

------

## Support Packets

### error

This packet is returned when error conditions arise.  It has the following form:

```
    ({
        (string)  "error",
        (int)     5,
        (string)  originator_mudname,
        (string)  0,
        (string)  target_mudname,
        (string)  target_username,
        (string)  error_code,
        (string)  error_message,
        (mixed *) error_packet
    })
```

The error_code values are standardized and are summarized in the [Error Summary](https://web.archive.org/web/20070813072129/http://www.intermud.org/i3/protocol.php3#errors) section.  The error_message does not follow any particular standards (yet?), but should be a message that can be displayed to a user.  The error_packet may contain the packet that caused the particular error to be generated.  If the packet is unavailable (i.e. due to code structure), then the value 0 may be used.

### startup-req-3

This packet is delivered to a mud's router when the mud first establishes the connection.  If the mud has never received a password from a server, it should send 0.  The server is responsible for creating a random password for new muds, and validating the password of a mud before allowing a mud to connect from a site other than the one from which it normally connects.

A startup-req-3 packet has the following form:

```
    ({
        (string)  "startup-req-3",
        (int)     5,
        (string)  originator_mudname,
        (string)  0,
        (string)  target_mudname,         // the router
        (string)  0,

        (int)     password,
        (int)     old_mudlist_id,
        (int)     old_chanlist_id,

        // these correspond to the values in a mudlist info_mapping
        (int)     player_port,
        (int)     imud_tcp_port,
        (int)     imud_udp_port,
        (string)  mudlib,
        (string)  base_mudlib,
        (string)  driver,
        (string)  mud_type,
        (string)  open_status,
        (string)  admin_email,
        (mapping) services
        (mapping) other_data
    })
```

The -3 on the packet type indicates that the mud will use Protocol Version 3 for communication (this specification).  Future changes in the protocol can update this number as required.  The router network must support all protocol versions and must translate packets between muds with different protocol versions.  Error packets will be returned to a mud if that mud attempts to use a service that cannot be translated (e.g. services that are only available in later protocol versions).

*Note: version 2 of the protocol had a smaller [locate-reply](https://web.archive.org/web/20070813072129/http://www.intermud.org/i3/protocol.php3#locate) packet.  No other changes were made to the specification.*  

*Note: version 1 of the protocol was almost exactly the same as this version except that it was missing a couple fields from the startup packet and the corresponding info_mapping in the mudlist packet.*  

All pieces of information are required to be sent to a router except for the port numbers and other_data.  The player_port may be 0 if the mud is private/closed.  The OOB ports may be 0 if the services that require them will not be provided by the mud. other_data may be 0 if a mud has no "other data" (see below).

open_status is a string describing the current status of the mud.  Suggested (strongly encouraged values) are:

- "mudlib development"
- "beta testing"
- "open for public"
- "restricted access"

mud_type specifies the type/family/genre of the mud driver.  Examples are: *LP*, *MOO*

services is a mapping with service names as keys.  The allowable services include those specified within this specification (in the [Services](https://web.archive.org/web/20070813072129/http://www.intermud.org/i3/protocol.php3#services) section); these simply have a value of 1 in the mapping.  In addition, extended (non-standard) services may be specified with service-specific information for their value. Here is the current list of services (keys and values) that a mud may make available along with some example extended services:

- "tell" : 1

- "emoteto" : 1

- "who" : 1

- "finger" : 1

- "locate" : 1

- "channel" : 1

- "news" : 1

- "mail" : 1

- "file" : 1

- "auth" : 1

- "ucache" : 1

- "smtp" : port-number

  Specifies the port number of a mud's SMTP mail interface. Note that Intermud mail is normally delivered via the OOB TCP port.

  

- "ftp" : port-number

  Specifies the port number of a mud's FTP service.  Note that Intermud file transfer is normally delivered via the OOB TCP port.

  

- "nntp" : port-number

  Specifies the port number of a mud's NNTP server.  Note that Intermud news transfer is normally delivered via the OOB TCP port.

  

- "http" : port-number

  Specifies the port number of a mud's WWW server (httpd).

  

- "rcp" : port-number

  Specifies the port number of a mud's Remote Creator server (currently a facility provided by Nightmare Lib IV). 

  

- "amcp" : version-string
   Indicates the mud supports AMCP of some particular version.

Additional services and service-specific data may be added in the future as required. 

The other_data field (when provided) contains a mapping with string keys. The values are arbitrary and determined by the key.  These key/value pairs are used to specify information that is not related directly to the I3 network operation.  At this time, there are no defined keys in this specification or via precedent.  Individual muds are free to define their own key/value pairs.  It is highly recommended that attempts be made to avoid namespace collisions.  For example, if the Lima Mudlib decides to place an entry into the other_data field, it might use a key of "lima-somekey".

### startup-reply

This packet will be delivered to a mud for three conditions: in response to a startup-req packet, when the router wishes the mud to connect to a different router, or when the set of routers change for some reason.

```
    ({
        (string)   "startup-reply",
        (int)      5,
        (string)   originator_mudname,     // the router
        (string)   0,
        (string)   target_mudname,
        (string)   0,
        (string *) router_list,
        (int)      password
    })
```

The router_list is an array representing an ordered list of routers to use.  The first element should be the router that the mud should use.  Typically, this will be the router that the mud initially connected to. If not, however, then the mud should close the connection and reopen to the designated router.  The list should be saved and used in case of failure to connect to a router. Each element in the list is an array of two elements; the first element is the router name, the second element is the router's address in the following format: "ip.ad.re.ss portnum".  Note that this address can be passed to MudOS's socket_connect() function. For example: `({ "*nightmare", "199.199.122.10 9000" })`.

The first router specified in the list will be the mud's preferred router.  Future initial connections and startup-req packets will go to that router until told otherwise.

### shutdown

This packet is delivered to a mud's router when the mud is gracefully shutting down. It has the following form:

```
    ({
        (string) "shutdown",
        (int)    5,
        (string) originator_mudname,
        (string) 0,
        (string) target_mudname,         // the router
        (string) 0,
        (int)    restart_delay
    })
```

restart_delay can be used to specify when the mud thinks it will be restarted.  This value is measured in seconds. 0 may be used to mean unknown/indefinite.  If a mud will be back "immediately," then it can simply use 1 second for this.

If the restart_delay is greater than 5 minutes, then the router will mark the mud as "down" immediately rather waiting the 5 minutes. Likewise, if the restart_delay specifies a duration longer than 7 days, the mud will be deleted from the Intermud.  Any time less than 5 minutes will cause the router to operate as normal: it will wait for a while.

### mudlist

The router will send this to a mud whenever the mud's list needs to be updated.  Typically, this will happen once right after login (based on the old_mudlist_id that a mud provided in the [startup-req-3](https://web.archive.org/web/20070813072129/http://www.intermud.org/i3/protocol.php3#startup-req) packet), and then as changes occur within the intermud network. A mud should remember the mudlist and its associated mudlist_id across reconnects to the router.

```
    ({
        (string)  "mudlist",
        (int)     5,
        (string)  originator_mudname,     // the router
        (string)  0,
        (string)  target_mudname,
        (string)  0,
        (int)     mudlist_id
        (mapping) info_mapping
    })
```

The info_mapping contains mud names as keys and information about each mud as the value.  This information is specified as an array with the following format:

```
    ({
        (int)     state,
        (string)  ip_addr,
        (int)     player_port,
        (int)     imud_tcp_port,
        (int)     imud_udp_port,
        (string)  mudlib,
        (string)  base_mudlib,
        (string)  driver,
        (string)  mud_type,
        (string)  open_status,
        (string)  admin_email,
        (mapping) services
        (mapping) other_data
    })
```

Each record of information should replace any prior record for a particular mud.  If the mapping's value is zero, then the mud has been deleted (it went down and has not come back for a week) from the Intermud.

state is an integer with the following values:

```
        -1  mud is up
         0  mud is down
         n  mud will be down for n seconds
```

### oob-req

\### not sure on this packet... discussion required ### 

This packet is delivered to a target mud when an originating mud wishes to connect to its OOB port.  This packet should only be delivered if the target mud does not support the [auth](https://web.archive.org/web/20070813072129/http://www.intermud.org/i3/protocol.php3#auth) service or if the service using the OOB connection will not require use of the auth service. 

The packet has the following form:

```
  ({
        (string) "oob-req",
        (int)    5,
        (string) originator_mudname,
        (string) 0,
        (string) target_mudname,
        (string) 0,
   })
```

The originating mud must then connect within 10 minutes.  The target mud may use this opportunity to actual open the OOB port and listen for the incoming connection request.

### oob-begin

This packet is used over an OOB link (see [OOB Protocols](https://web.archive.org/web/20070813072129/http://www.intermud.org/i3/protocol.php3#oob)). It is used to specify the authorization information for this connection. 

Since this packet operates over an OOB link, it does not need to conform to standard packet formats.  Its format is:

```
  ({
        (string) "oob-begin",
        (string) originator_mudname,
        (int)    auth_type,
        (int)    auth_token
   })
```

The auth_type will contain one of the following values:

```
        0 no authentication used
        1 auth-mud-req used
```

### oob-end

This packet is used over an OOB link (see [OOB Protocols](https://web.archive.org/web/20070813072129/http://www.intermud.org/i3/protocol.php3#oob)). It is used to signify that a mud is done delivering packets to the other mud. 

Since this packet operates over an OOB link, it does not need to conform to standard packet formats.  Its format is:

```
  ({
        (string) "oob-end",
        (string) mudname
   })
```

mudname states who has completing delivering packets.  This is used to simplify the process of determining which mud (of a possible many outstanding OOB connections) just completed their work.  This could be done simply by matching the socket which received the data to a record of what mud it connected to, but specifying the mud should make this process simpler for some clients.

------

## OOB Protocols

\### more to come ### 



1.  Originator uses target's [auth](https://web.archive.org/web/20070813072129/http://www.intermud.org/i3/protocol.php3#auth) service to fetch necessary authorization tokens.  If authorization is not needed, then an [oob-req](https://web.archive.org/web/20070813072129/http://www.intermud.org/i3/protocol.php3#oob-req) packet is sent.  This step operates of the in-band network.
2.  Originator connects to target's OOB port.
3.  Originator delivers an [oob-begin](https://web.archive.org/web/20070813072129/http://www.intermud.org/i3/protocol.php3#oob-begin) packet, containing the authorization tokens, over the OOB link to the target mud.
4.  The target mud validates the authorization tokens.  If the validation fails, then the target mud closes the connection.
5.  Target returns an [oob-begin](https://web.archive.org/web/20070813072129/http://www.intermud.org/i3/protocol.php3#oob-begin) packet (with empty authorization information) to tell the originator to begin.
6.  Originator delivers all queued packets to the target mud.  The target should respond with various replies and acknowledgements during this process.  The target is not yet allowed to actively send packets yet.
7.  Originator delivers an [oob-end](https://web.archive.org/web/20070813072129/http://www.intermud.org/i3/protocol.php3#oob-end) packet, signalling completion.
8.  If the target mud has any packets to deliver in an outbound queue for the originator, then it performs the deliveries.  The originator should respond with various replies and acknowledgements during this process.
9.  The target mud delivers an *oob-end* packet if no deliveries are to be made, or upon completion of the deliveries.
10.  If the originator has futher packets to deliver (some may have been placed in the originator's outbound queue during this process), then the process returns to step #6.
11.  The originator drops the connection.

The target mud may time out and close the connection if it does not receive any packets for 10 minutes.

------

## Router Design

The routers will create, open, and maintain MUD mode TCP sessions with each of the other routers. Each router will also maintain a list of the status of all links in the router network. This link information will be used for routing around a failed link until the link is reestablished.

Each router will maintain a complete list of all muds on the Intermud, the information associated with each one, the up/down/rebooting state, and which router the mud is connected to.

Each router will also maintain a list of information about each Intermud channel, including the channel host/owner, the type of channel, and the list of admitted/banned muds for the channel.



### List Synchronization

The two sets of lists are synchronized with the same scheme from a mud's point of view: they receive a unique token that precisely denotes the state of their list. The router network can remember how to provide deltas from one token to another or can just deliever complete lists when a request arrives from a mud that needs an update.

To guarantee uniqueness of the token, the routers can use the current time.  This becomes complex because all routers must use the same token for the list (in case a mud is switched to a new router, it should not have to fetch the whole list). Using the time across multiple machines with different time drifts and within different time zones and accounting for poor system administration can make the problem quite difficult. A scheme is required to meet each of these problems.

Assume a steady state where each router has the same token and a consistent list with respect to the other routers. Now, let us say a router generates a delta to the list.  It will create a new token as `max(old_token + 1, time())`. This token will be passed with the list delta to all other routers. The other routers will install the change, record the new token, and propogate the information to their connected muds. Within the router network, the router that originated the current token is remembered.  This is needed to uniquely identify deltas that occurred at the same instant.  Using the above formula for creating the token also ensures that a router will generate unique tokens even if they occur within the same second (the second token will be 1 more than the first).

Now, what happens when two routers generate deltas at about the same time?  Call the tokens `t1` and `t2` and they are ordered as `t1 < t2`. There are two situations that will occur: a router will receive t1 before t2, or vice-versa.  First, the stable-state token must be specified for this situation (it cannot simply be the last token to arrive; otherwise, some routers will have t1 and some will have t2).  The rule is simply: `final_token = max(t1,t2)`.

Now, if a router receives t1 before t2, then everything will be fine - it will have notified its muds with the t1-delta then with the t2-delta.  The muds and the router will become stable with a current token of t2.

However, if the router receives t2 first, then there will be problems.  It will notify the muds with the t2-delta, then with the t1-delta.  The router will deliver a token of t2 with the t1-delta change to the muds and will stabilize with that token. This would seem to be okay unless a mud disconnected between the t2-delta and t1-delta notifications.  When it reconnects, it will ask for changes since t2 and receive none.  Simply put: the t1-delta is subverted to using the t2 token and therefore does not uniquely identify the change.

This problem is solved with the use of a "alter-token" packet.  The t1-delta gets a new token applied to it and is recirculated through the router network.  Since many routers may see the conflict and initiate an alter-token operation, there should be a way to eliminate duplicates; otherwise, two altered-t1-deltas could again create a synchronization problem.  To overcome this, the altered token will be set to t2+1.  The routers will ignore the receipt of an altered-token-delta if its current token came from a similar altered-token.

\### todo: two packets with same time; multiple packets? - solve case of two - inductance handles rest ###



```
    ({
        (string)  "XXXlist-delta",
        (int)     5,
        (string)  originator_mudname,     // the router
        (string)  0,
        (string)  "*",                    // router broadcast
        (string)  0,
        (int)     token,
        (mapping) list_delta_info
    })
    ({
        (string)  "XXXlist-altered",
        (int)     5,
        (string)  originator_mudname,     // the router
        (string)  0,
        (string)  "*",                    // router broadcast
        (string)  0,
        (int)     altered_token,
        (mapping) list_delta_info
    })
```

### Packet Routing

To efficiently implement the routing, the routers will route based entirely on the originator/target fields of the packet.  Knowledge of the packet types will not be required.  This provides flexibility for extensions and, most likely, increased routing speed.

Rules for routing are based entirely on the target_mudname.  If it is provided, then the packet will be sent to the mud if it is connected to the receiving router.  Otherwise, the router for the mud is looked up and the packet is forwarded there. If the mudname refers to the router itself, then the packet will be passed "up" to higher-level router processes.

\### info on routing around downed links ###



------

## Other Drivers/Mudlibs

There will exist two reference implementations of the mudlib side of the protocol.  These can be ported/used to create implementations for alternate mudlibs and for MudOS pre-v21 drivers.

For drivers that have TCP but not MUD-mode, they will need to parse the incoming transmissions (MUD-mode is effectively a combination of save_variable() and a standard TCP socket).

For drivers without TCP sockets, then a gateway will need to be written to hook into the router network and gateway the protocol between the MUD-mode TCP sockets and, say, UDP.

Gateways can also be used for non-LP based muds (e.g MOO, MUCK, Diku, etc).

Note that it is possible for the router network protocol to evolve independently of the protocol used by the muds.  This could involve moving away from the MUD-mode style of communication.



------

## Error Summary

There are a "standard" set of error_codes that are returned by routers and remote muds in [error](https://web.archive.org/web/20070813072129/http://www.intermud.org/i3/protocol.php3#error) packets. The error_message is not as well defined at this point, but should be something that can be displayed to the user.

Following is a list of the standard error codes returned by the routers:

- unk-dst : unknown destination mud
- not-imp : feature not yet implemented
- unk-type : unknown packet type (also sent by muds)
- unk-src : unknown source of packet (unregistered mud)
- bad-pkt : bad packet format (also sent by muds)
- bad-proto : protocol violation (packet used incorrectly, at the wrong time, etc)
- not-allowed : operation not allowed (e.g. channel bans)

Following is a list of standardized error codes that a mud may return:

- unk-type : unknown packet type (also sent by router)
- unk-user : unknown target user
- unk-channel : unknown channel name
- bad-pkt : bad packet format (also sent by router)

------

## Packet Types Summary

- [tell](https://web.archive.org/web/20070813072129/http://www.intermud.org/i3/protocol.php3#tell) : send a message to a remote user
- [emoteto](https://web.archive.org/web/20070813072129/http://www.intermud.org/i3/protocol.php3#emoteto) : send an emote to a remote user
- [who-req](https://web.archive.org/web/20070813072129/http://www.intermud.org/i3/protocol.php3#who) : request a list of users on a remote mud
- [who-reply](https://web.archive.org/web/20070813072129/http://www.intermud.org/i3/protocol.php3#who) : reply with a list of users on a remote mud
- [finger-req](https://web.archive.org/web/20070813072129/http://www.intermud.org/i3/protocol.php3#finger) : request finger information for a remote user
- [finger-reply](https://web.archive.org/web/20070813072129/http://www.intermud.org/i3/protocol.php3#finger) : reply with finger information for a remote user
- [locate-req](https://web.archive.org/web/20070813072129/http://www.intermud.org/i3/protocol.php3#locate) : request the location of a user
- [locate-reply](https://web.archive.org/web/20070813072129/http://www.intermud.org/i3/protocol.php3#locate) : reply with the location of a user
- [chanlist-reply](https://web.archive.org/web/20070813072129/http://www.intermud.org/i3/protocol.php3#chanlist-reply) : reply with/update the list of available channels
- [channel-m](https://web.archive.org/web/20070813072129/http://www.intermud.org/i3/protocol.php3#channel-m) : send a message over a channel
- [channel-e](https://web.archive.org/web/20070813072129/http://www.intermud.org/i3/protocol.php3#channel-e) : send an emote over a channel
- [channel-t](https://web.archive.org/web/20070813072129/http://www.intermud.org/i3/protocol.php3#channel-t) : send a targetted emote over a channel
- [channel-add](https://web.archive.org/web/20070813072129/http://www.intermud.org/i3/protocol.php3#channel-add) : register a new channel
- [channel-remove](https://web.archive.org/web/20070813072129/http://www.intermud.org/i3/protocol.php3#channel-remove) : remove a channel from name server databases.
- [channel-admin](https://web.archive.org/web/20070813072129/http://www.intermud.org/i3/protocol.php3#channel-admin) : adminstrate the participants of a channel
- [chan-filter-req](https://web.archive.org/web/20070813072129/http://www.intermud.org/i3/protocol.php3#chan-filter-req) : filter a channel
- [chan-filter-reply](https://web.archive.org/web/20070813072129/http://www.intermud.org/i3/protocol.php3#chan-filter-reply) : return filtered channel messages to the router
- [chan-who-req](https://web.archive.org/web/20070813072129/http://www.intermud.org/i3/protocol.php3#chan-who-req) : request a who list for people listening to a channel on a remote mud
- [chan-who-reply](https://web.archive.org/web/20070813072129/http://www.intermud.org/i3/protocol.php3#chan-who-reply) : return a requested channel who list
- [channel-listen](https://web.archive.org/web/20070813072129/http://www.intermud.org/i3/protocol.php3#channel-listen) : tune a mud as a whole into or out of a channel
- [chan-user-req](https://web.archive.org/web/20070813072129/http://www.intermud.org/i3/protocol.php3#chan-user-req) : request channel user's info
- [chan-user-reply](https://web.archive.org/web/20070813072129/http://www.intermud.org/i3/protocol.php3#chan-user-reply) : reply with channel user's info
- [news-read-req](https://web.archive.org/web/20070813072129/http://www.intermud.org/i3/protocol.php3#news) : retrieve a news post from an OOB news server.
- [news-post-req](https://web.archive.org/web/20070813072129/http://www.intermud.org/i3/protocol.php3#news-post-req) : post news to an OOB news server.
- [news-grplist-req](https://web.archive.org/web/20070813072129/http://www.intermud.org/i3/protocol.php3#news-grplist-req)
- [mail](https://web.archive.org/web/20070813072129/http://www.intermud.org/i3/protocol.php3#mail) : deliver an item of mail
- [mail-ack](https://web.archive.org/web/20070813072129/http://www.intermud.org/i3/protocol.php3#mail-ack) : acknoledge a mail
- [file-list-req](https://web.archive.org/web/20070813072129/http://www.intermud.org/i3/protocol.php3#file-list-req) : request a list of files on a remote mud
- [file-list-reply](https://web.archive.org/web/20070813072129/http://www.intermud.org/i3/protocol.php3#file-list-reply) : reply with list list
- [file-put](https://web.archive.org/web/20070813072129/http://www.intermud.org/i3/protocol.php3#file-put) : send a file to a remote mud
- [file-put-ack](https://web.archive.org/web/20070813072129/http://www.intermud.org/i3/protocol.php3#file-put-ack) : acknoledge a file-put
- [file-get-req](https://web.archive.org/web/20070813072129/http://www.intermud.org/i3/protocol.php3#file-get-req) : get a file from a remote mud
- [file-get-reply](https://web.archive.org/web/20070813072129/http://www.intermud.org/i3/protocol.php3#file-get-reply) : reply with the requested file
- [auth-mud-req](https://web.archive.org/web/20070813072129/http://www.intermud.org/i3/protocol.php3#auth-mud-req) : request a mud-level authorization token
- [auth-mud-reply](https://web.archive.org/web/20070813072129/http://www.intermud.org/i3/protocol.php3#auth-mud-reply) : reply with a mud-level authorization token
- [ucache-update](https://web.archive.org/web/20070813072129/http://www.intermud.org/i3/protocol.php3#ucache) : update cached user information
- [error](https://web.archive.org/web/20070813072129/http://www.intermud.org/i3/protocol.php3#error) : provide error information
- [startup-req-3](https://web.archive.org/web/20070813072129/http://www.intermud.org/i3/protocol.php3#startup-req) : provide start up information to the router
- [startup-reply](https://web.archive.org/web/20070813072129/http://www.intermud.org/i3/protocol.php3#startup-reply) : reply with/update the basic startup information (router list)
- [shutdown](https://web.archive.org/web/20070813072129/http://www.intermud.org/i3/protocol.php3#shutdown) : gracefully indicate shutdown
- [mudlist](https://web.archive.org/web/20070813072129/http://www.intermud.org/i3/protocol.php3#mudlist) : reply with/update the list of available muds
- [oob-req](https://web.archive.org/web/20070813072129/http://www.intermud.org/i3/protocol.php3#oob-req) : request setup for OOB connection
- [oob-begin](https://web.archive.org/web/20070813072129/http://www.intermud.org/i3/protocol.php3#oob-begin) : begin OOB communication
- [oob-end](https://web.archive.org/web/20070813072129/http://www.intermud.org/i3/protocol.php3#oob-end) : end one side of an OOB process

------

## Compressed Mode

\### to be filled in with specifics ###

Generally, many of the fields will be replaced with numbers rather than full strings.  Some of the fields can be paired up using various combined keys. For example, originator_mudname can almost always be ommitted since it can be inferred from the TCP session at the router end.  Also, all the request/reply pairs can use a request key rather than recording actual user names (the user is associated with the request key on the originating mud). 



------

## Change Log

-  Deathblade, February 15
  -  added section for contributors and implementors
  -  removed the section on reference implementations
  -  cleared up a couple OOB items
-  Winddle, January 7
  -  update file, removing local_fname and references to /ftp to remove some confusion.
-  Deathblade, December 20
  -  add this change log
  -  update to startup-req-3 for the change in the locate-reply packet
-  Winddle, December 6
  -  minor fix to mail
  -  fix TTL to be int in chan-filter-reply
-  Winddle, October 20
  -  update to mail
  -  update to file
  -  fixed a few 'broken' links
-  Deathblade, October 7
  -  update to startup-req-2
  -  modify the startup-req-2 packet and the mudinfo records to include    "admin_email" and "other_data"
  -  add amcp service
  -  strip really old changelog entries
-  Winddle, September 24
  -  add typecasting
-  Deathblade, September 10
  -  add some OOB from Winddle
  -  remove auth-user-xxx
-  Deathblade, August 9
  -  added OOB information
-  Deathblade, August 8
  -  add the auth, emoteto, ucache services
  -  add section for OOB discussion
  -  added a couple more standard errors
  -  renamed some news packets to fit within the "news name space"
  -  add comments to mail, file, and news protos re: auth service
