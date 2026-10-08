---
sidebar_label: Zebedee
---
:::note
**Source**: [http://mud.stack.nl/intermud/zebedee.protocol.html](https://web.archive.org/web/20051229142240/http://mud.stack.nl/intermud/i2.protocol.html)
Also: [Zebedee Inetd Services](https://web.archive.org/web/20011218124054/http://mud.stack.nl/intermud/zebedee.html)
Also: [Portals](https://groups.google.com/g/de.alt.mud/c/UR0xHq0YARc/m/HywTEp1LMgoJ) and used [source](https://github.com/unitopia-de/lp245.git)

The CD protocol was quite basic and easy to implement, however it has some important drawbacks: It is not possible to send messages longer than 1kb. Only 1 intermud channel is implemented. And there is no message acknowledgement - even though many packets are lost due to the unreliability of the UDP (user datagram protocol) used. So the protocol is not really fit for sending mud-lists, mail or files.

All flaws of the CD-implementation are taken care of in it the new intermud protocol distributed as the "Inetd" package. This was created by **Nostradamus@Zebedee** (Mark Lewis). Here a message can be send in several parts and if no acknowledgement is received, a message is send again (up to 3 times). The inetd distribution can be easily added in many LPC mudlibs although it was primary designed for Amylaar (with CD compability in mind!)

A great extension providing intermud-mail is included in the latest distributions. This mail protocol was developed on top of Zebedee's implementation by **Alvin@Sushi** (James W. Armstrong). The latest (and most likely last ever) version is Inetd v0.7 beta 2, which was released in September 1996(?) The first implementation of this protocol was available late 1993 (maybe even earlier).
:::

# Zebedee Intermud

```mermaid
architecture-beta
    service m1(server)[MUD 1]
    service m2(server)[MUD 2]
    service m3(server)[MUD 3]
    service m4(server)[MUD 4]

    m1:R -- L:m2
    m1:B -- T:m3
    m2:B -- T:m4
    m3:R -- L:m4
    m1:R -- L:m4
    m2:L -- R:m3
```

## Initializing
On start-up the MUD that wishes to connect must load a static [MUD list](#the-mud-list-format) from a well known source (e.g. file system). Within this list there a known peers for Zebedee communication.





This file was originally written as a brief outline of the intermud protocol for use by developers interested in incorperating similar, compatible intermud protocols into their own mud systems. It is included here as it provides a much more detailed description of the intermud protocol than that provided by the original PROTOCOL file, and hence may be of use to LpMud developers.

## PACKET PROTOCOL / FORMAT

All information is transferred as a string via a UDP port (each mud has 1 send and 1 receive port). This kindof transfer is inherently unreliable, but it's fast and doesn't use up file descriptors. The format of the strings (packets) is as follows:

```
header1:body1|headerN:bodyN|DATA:body-data
```

In other words, a header name, followed by a : and then the data associated with this header. Each header/body pair is separated by the | character. This means that headers and their body cannot contain the | character. You should check for this in outgoing packets to aviod decoding errors at the recieving end. The exception to this is the DATA field. If it is present, it is ALWAYS positioned at the end of the packet. Once a DATA header is found, everything following it is interpreted as the body of the DATA field. This means it can contain special characters without error and it is used to carry the main body or data of all packets.

By convention, predefined system fields will use capital letters for field headers and custom headers used by specific applications will use lowercase names to avoid clashes. The defined system fields are generally refered to by a set of macros which are defined in a common header file for clarity.

There is one exception to this header format; If the data is too large to be transmitted in one single packet, it will be split into packets of convenient size, each with a special unique packet header to enable them to be reassembled at the receiving end. These headers are of the format:

```
PKT:mudname:packet-id:packet-number/total-packets|rest-of-packet
```

In this case, the mudname and packet-id combine to form a unique id for the packet. The packet-number and total-packets information is used to determine when all buffered packets have been received. The rest-of-packet part is not parsed, but is stored while the receiver awaits the other parts of the packet. When/if all parts have been received they are concatenated and decoded as a normal packet.

### Additional format
With release of version 0.7 Beta 2, an alternative package format was introduced, where the delimiter "|" was replaced with a linebreak.

:::note[Developers note for 0.7 Beta 2]
Developers please note: This code begins the migration to the new packet format (which resembles SMTP). The packet format described elsewhere in this documentation will become obsolete. This code still transmits packets in the old format by default, but reads both the old and new formats. The new format will be of the form:
```
HEADER1:data1
HEADERN:dataN

body-data
```
In other words, the old '|' delimiter will be replaced by '\n', and the main DATA field will now be preceeded with a blank line ("\n\n") rather than "DATA:".
:::
As of today existing implementations still use the old format. They may understand the new one, though.

------

## PACKET ENCODING / DECODING

Only 2 generic data types are fully suported within the inetd code itself (namely strings and integers), though others can easily be used by converting them to one of the supported data types before transfer and converting back again in receipt. The LpMud "object" data type is converted to a string automatically by the inetd on encoding, but no such conversion is carried out on decoding.

On encoding integers are simply converted to a corresponding string. Strings are left untouched as long as there is no ambiguity as to wether they should be decoded as a string or an integer. In this case of ambiguity, the string is prepended with a $ character. If the first character of a string is the $ character, it is escaped by prepending another $ character. On decoding, any string with a $ as its first character will have it removed and will then be treated as a string. Any remaining strings that can be converted to an integer and then back to a string with no loss of information are considered to be integers. Any remaining strings are treated as such and are left unaltered.

------

## DEFINED SYSTEM HEADERS

- "RCPNT" (RECIPIENT)

  The body of this field should contiain the recipient the mesage is to be sent to if applicable.

- "REQ" (REQUEST)

  The name of the intermud request that is being made of the receiving mud. Standard requests that should be supported by all systems are "ping" (PING), "query" (QUERY), and "reply" (REPLY). The PING request is used to determine wether or not a mud is active. The QUERY request is used to query a remote mud for information about itself (look at the udp/query module for details of what information can be requested). The REPLY request is special in that it is the request name used for all replies made to by mud B to an initial request made by a mud A. It is mud A's responsibility to keep track of the original request type so that the reply can be handled appropriately.

- "SND" (SENDER)

  The name of the person or object which sent the request or to whom replies should be directed. This is essential if a reply is expected.

- "DATA" (DATA)

  This field should contain the main body of any packet. It is the only field that can contain special delimiting characters without error.

The following headers are used internally by the inetd and should not be used by external objects:

- "HST" (HOST)

  The IP address of the host from which a request was received. This is set by the receiving mud and is not contained in outgoing packets.

- "ID" (ID)

  The packet id. This field is simply an integer which is set by the sending inetd. The number is incremented each time a packet is sent (zero is never used). This field is only needed if a reply is expected. REPLY packets _must_ include the original request id. This is _not_ done by the inetd.

- "NAME" (NAME)

  The name of the local mud. Used for security checking and to update host list information.

- "PKT" (PACKET)

  A special header reserved for packets which have been split. See PACKET PROTOCOL / FORMAT.

- "UDP" (UDP_PORT)

  The UDP port the local mud is receiving on. Used for security checking and updating host list information.

- "SYS" (SYSTEM)

  Contains special system flags. The only system flag used at present is TIME_OUT. This is included in packets returned due to an expected reply timing out to differentiate it from an actual reply.

------

## UDP REQUESTS / MODULES

The following are standard request types that must be supported by all systems:

### "ping" (PING)

  This module should return a REPLY packet that contains the original requests ID in it's ID field and the SENDER in it's RECIPIENT field. It should also include an appropriate string in the DATA field, eg. "Mud-Name is alive.\n"

### "query" (QUERY)

  This module expects the type of query requested to appear in the recieved DATA field. It should return a REPLY packet containing the original ID in the ID field, the SENDER in it's RECIPIENT field, and the query type in a QUERY field. The DATA field should contain the information requested.

For details of how other intermud requests operate, look at the relevant module code.



## Zebedee's Inetd Services
A good protocol definition and implementation are available (from Nostradamus@Zebedee), however I didn't find a good list of available services and commands. So I created this list myself, based on the implementation.

Note that the fields "NAME" and "UDP_PORT" should be present in every message. Very common are the fields "ID" (used whenever an reply is expected) and "SND" (the sender: he should receive the reply). These fields will not be mentioned in the list below.

### channel
    The channel-request is used for sending a message on any channel. The "CMD" field is optional and may be omitted for normal messages. Note that you should not send an history or list request to _all_ known muds!
    
    "CHANNEL"
        The channel on which a message is send (the standard channels are "intermud", "intercode", "interadm", "d-chat", "d-code" and "d-adm"; on the d-channels German is spoken)
    
    "DATA"
        The message to be send (not used with history/list request)
    
    "CMD" (optional)
    
        ""
            for normal intermud messages,
    
        "emote"
            if the message is an emote,
    
        "history"
            for an history request: the last 20 lines of this channel will be shown.
    
        "list"
            to list all remote users listening to this channel 
    
    "EMOTE" (optional)
    
        1
            The message is a normal emote.
    
        2
            The message is a gemote. 

### finger
    Retreive information about a player or creator on a remote mud.
    
    "DATA"
        The player of whom information is requested 
    
    Extensions implemented by some MUDs (see "ANSI colours over Intermud" below):
    
    "ansi" (optional)
        The colour level the requesting player can actually see:
        "no", "2", "8", "16", "256", "truecolor" or "screenreader"
    
    "charset" (optional)
        "utf-8" or "ascii": whether the requesting player can display UTF-8

### locate
    Check whether a certain player is logged on at a remote mud. This request is usually send to all known muds at the same time.
    
    "user"
        Name of the person who requests the information.
    
    "vbs"
        The verbose option has only two pre-defined values:
    
        1
            Even report when the result was negative
    
        2
            Don't do timeouts, but keep waiting 
    
    "fnd"
        The found option is only used in the reply and it's value is either 1 (success) or 0 (failure). The absence of a found parameter indicates failure as well. 
    
    "DATA"
        The player to find. 

### man
    Retreive a manual page from a remote mud. Many muds don't support this feature...
    
    "DATA"
        The name of the requested manual page 

### mail
    An extension to the standard protocol, by Alvin@Sushi. This is used to send mails from one mud to another.
    
    "udpm_status"
        This field should only be used in the reply and indicates how mail is handled. Currently there are four pre-defined values for the status field:
    
        0
            time out
    
        1
            delivered ok
    
        2
            unknown player
    
        3
            in spool (will be delivered later) 
    
    "udpm_writer"
        Name of the person who wrote this mail
    
    "udpm_spool_name"
        Should be returned as sent, this value is used to remove the mail from the spool directory after it has been delivered (or refused)
    
    "udpm_subject"
        Subject of the mail message
    
    "DATA"
        The body of the mail (the actual message) 

### ping
    A ping request has only the standard fields, the reply is usually a short string like " is alive."

### query
    Get standard information about another mud. This is the only command of which the reply may not include a load of rubbish, but should only hold the requested information, so that it can be parsed by the server.
    
    "DATA"
        The following queries are pretty much standard:
    
        "commands"
            List all commands that are supported by the inetd
    
        "email"
            The email-address of the mud administrator(s)
    
        "hosts"
            A listing of all hosts in a special format [t.b.d.]
    
        "inetd"
            The version number of the inetd used
    
        "list"
            The list of all items which can be queried
    
        "info"
            A short human-readable string with practically "query" information
    
        "mud_port"
            The portnumber that players connect to on login
    
        "time"
            The local time for this mud
    
        "users"
            A list of the people that are active in this mud
    
        "version"
            The version of the mud-driver (and library)
    
        "www"
            The URL of the mud's web page (e.g. http://mud.stack.nl/) 



        Extensions implemented by some MUDs:

        "mud_port_tls"
            The portnumber to establish a TLS/SSL encrypted connection to this mud

        "encoding"
            The encoding this mud prefers for intermud packages
            Intermud communication should default to ASCII, since this is the only encoding guaranteed to work everywhere.
            If a MUD can handle UTF8 or any other more advanced encoding it can announce this capability here.
            Other MUDs sending requests can then upgrade the encoding accordingly.

        "hosts-json"
            Since the format of the 'hosts' query was never standardized there is quite some variety of formats in the wild.
            Some MUDs use different separators, different columns, or a different order of columns.
            The most commonly used separator ':' also clashes with IPv6 addresses (prompting some MUDs to use a different separator like ';').
            
            This query aims to rectify this, by sending the host list as a JSON array of objects.
            
            Mandatory fields are:
            ```
            {
                "name": the mud name,
                "ip": the mud's IP; if the MUD supports IPv4 and IPv6 this should be the v4 address for maximum compatibility,
                "udp_port": the port for UDP intermud communication,
            }
            ```
            
            Additional fields can be added if available:
            ```
            {
                "ip6": the MUD's IPv6 address (additionally to IPv4 in the "ip" field)
                "mud_port": the port for telnet connections,
                "mud_port_tls": the port for TLS/SSL connections,
                "udp_encoding": the MUD's preferred encoding for intermud communication,
                "commands": [ an array of supported commands ],
                "queries": [ an array of supported queries ],
                "last_contact": the unixtime this mud was last seen,
            }
            ```
            
        "mssp-json"
            The MUD's MSSP data as JSON object.

        "ansi"
            The MUD's baseline for ANSI colour codes in anything its players receive over intermud:
            "no", "2", "8", "16", "256" or "truecolor". No reply means "no".
            See "ANSI colours over Intermud" below.

### reply
    This request method is used for _all_ replies.
    
    "DATA"
        A human-readable string, containing the reply to a given query
    
    "RCPNT"
        The same name as in the "SND" field or the query; Usually this is the name of the player who initiated the query
    
    "QUERY"
        This field is only used in a response to a "query" request and should be equal to the "DATA" field of that request
    
    "VBS"
        This field is only used in a response to a "locate" request and should be equal to the "VBS" field of that request
    
    "FND"
        This field is only used in a response to a "locate" request and should be 1 if the player was located and 0 otherwise 

### tell
    Say something to a player on another mud.
    
    "RCPNT"
        Name of the player to whom you are talking
    
    "DATA"
        Whatever you wish to say to this person 

### who
    List the people that are active on a remote mud. The anwer usually contains some active information about the players, like titles, levels or age.
    
    "DATA"
        Not supported by many muds. Introduced August 1997.
        Additional switch(es) (space separated) that change the appearence of the resulting list. The switches normally resemble the switches used inside of that mud for the 'who' command. Typical values include:
    
        "short" "s" "-short" "-s" "kurz"
            Return a concise listing. 
        "alpha" "a" "alphabetisch" "-alpha" "-a"
            Sort the players alphabetically. 

## ANSI colours over Intermud (query ansi)

An extension implemented by Midgard and Beutelland in October 2026. It is backwards compatible: MUDs that don't support it ignore the extra fields and simply receive no colour content. A detailed description with example code for the common inetd (MorgenGrauen mudlib) is available at [midgardmud.de/intermud/ansi.html](https://midgardmud.de/intermud/ansi.html) (English and German).

### Motivation

Replies to intermud requests, most notably `finger`, may contain ANSI colour codes, for example a character portrait made of half-block characters (▀) with 24-bit foreground and background colours. The replying MUD, however, knows nothing about the player at the other end: whether their client handles 24-bit, 256 or only 16 colours, whether it understands UTF-8, or whether the requesting MUD passes colour codes through at all. Between existing MUDs this has caused:

- blinking text, because a filter that doesn't know `38;2;r;g;b` reads the numbers one by one (`38;2;5;0;255` contains a `5`: blink),
- grey or white areas where unknown codes were replaced by the default colour,
- question marks instead of pictures, because `▀` doesn't survive a conversion to ASCII.

### Principle

In the end the player's terminal decides, and only the MUD the player is connected to knows it. The extension therefore has two levels:

1. **Per request:** the requesting MUD tells the replying MUD what its player can see (fields `ansi` and `charset` in the request). This always takes precedence.
2. **MUD-wide:** `query ansi` returns a MUD's baseline, which applies when nothing is known about the individual player.

As with other non-standard fields, the field names are lower case (compare the `udpm_` fields of intermud mail).

### query ansi

A MUD asks another with `REQ:query` and `DATA:ansi`. The reply (`REQ:reply`, `QUERY:ansi`) carries the MUD's baseline in `DATA`: the colour level that reliably reaches its players in anything they receive over intermud (`finger`, but also `tell` or mail) when nothing is known about the individual player. It should not be higher than what the MUD itself passes through to its players. No reply means `no`. Supporting MUDs also list `ansi` in their reply to `query list`.

The query is modelled on `query encoding`. A MUD that sends colourful content asks at startup and whenever a new MUD appears, and remembers the answer, for example as an additional column in its host list. A missing reply must not cause a MUD to be marked as down.

### Fields ansi and charset in finger requests

A `finger` request may carry two optional fields describing the requesting player:

- `ansi`: the colour level that really reaches the player, i.e. the lower of what the player's terminal can do and what the requesting MUD passes through. Values as below, plus `screenreader`.
- `charset`: `utf-8` or `ascii`.

Possible sources for the colour level are MTTS (bit 256: truecolor, bit 8: 256 colours, bit 1: ANSI), a terminal type containing `256color`, the MUD's own web client, or a player setting. A player setting is useful because clients sometimes overestimate themselves, for example tintin++ inside GNU screen. If the level is unknown, the field should be left out rather than guessed; the requesting MUD's answer to `query ansi` then applies.

### Values

Each level includes the ones below it.

| Value | Meaning | Allowed SGR codes |
|---|---|---|
| `no` | no colours, no codes | none |
| `2` | monochrome, highlighting only | `1` (bold/bright), `4` (underline), `7` (inverse); reset with `0`, `22`, `24`, `27` |
| `8` | the eight basic colours | additionally `30`–`37`, `40`–`47` |
| `16` | plus the bright colours | additionally `90`–`97`, `100`–`107` |
| `256` | 256-colour palette | additionally `38;5;n`, `48;5;n` |
| `truecolor` | 24 bit | additionally `38;2;r;g;b`, `48;2;r;g;b` |
| `screenreader` | player using a screen reader: plain text only, no pictures, no colours and no frames, lines or boxes. Only in the per-request `ansi` field, never as a reply to `query ansi`. | none |

- Blinking (`5`) deliberately belongs to no level.
- For bright colours, senders should use `90`–`97` and `100`–`107` rather than `1`: depending on the terminal, bold is shown brighter, only bold or not at all, and there is no equivalent for the background.
- Values are strings, including `2`, `8`, `16` and `256`. Following the packet encoding rules above, digit-only strings are sent with a `$` prefix (the common inetd's `encode()` does this), otherwise they arrive as integers. Receivers should accept both.

### Choosing what to send

A MUD that sends colourful content in a reply decides in this order:

1. The `ansi` and `charset` fields of the request, if present.
2. Otherwise the requesting MUD's reply to `query ansi`.
3. Otherwise `no`: nothing colourful.

Without UTF-8, that is with `charset:ascii` or when the requesting MUD doesn't report UTF-8 for `query encoding`, only characters that survive any character set should be used, for example spaces with a background colour instead of half blocks.

### Example packets

```
# Midgard asks Beutelland for its baseline
REQ:query|NAME:Midgard|UDP:4246|ID:5|DATA:ansi

# Beutelland replies: 256 colours (digits with a $ prefix)
REQ:reply|NAME:Beutelland|UDP:4246|ID:5|QUERY:ansi|DATA:$256

# A player in Beutelland fingers eirik@Midgard;
# their client handles 256 colours and UTF-8
REQ:finger|NAME:Beutelland|UDP:4246|ID:17|SND:invisible|ansi:$256|charset:utf-8|DATA:eirik
```

### Security

- A MUD receiving replies for its players should pass through only SGR sequences (`ESC [`, then digits, `;` and `:`, then `m`) and remove every other escape sequence: other CSI commands such as cursor movement or clearing the screen, OSC sequences (for example window titles or links) and anything else following `ESC`, as well as control characters other than newline and tab. Otherwise a foreign MUD could alter the player's screen or, with status requests such as `ESC [ 6 n`, make the player's terminal send input to the MUD.
- Replies go to the source address of a UDP packet, which can be forged. A MUD sending large replies (a picture can take dozens of packets) should limit how many it sends per minute, so that `finger` cannot be abused as an amplifier.

### Backwards compatibility

The common inetd hands unknown fields such as `ansi` and `charset` to the request modules along with the other data, and modules that don't know them ignore them. MUDs that don't answer `query ansi` receive no colour content, which is the safe default. There is no new service and no change to the packet format.

### Implementations

| MUD | Reply to query ansi | Sends with finger | Serves colour content |
|---|---|---|---|
| Midgard | `256` | `ansi` (when the terminal is known; `screenreader` for players in plain-text mode) and `charset` | character portraits in truecolor, 256, 16 and 8 colours and as a spaces-only ASCII version; plain text for `screenreader` |
| Beutelland | `256` | `ansi` | – |

The idea of `query ansi` and of per-player information came from Invisible@Beutelland; Midgard worked out the fields and the picture versions.

## The MUD list format

A typical MUD list - here an example of active MUDs in July 2025 - looks like this:

```
AbendDaemmerung:178.254.21.129:4246:channel,finger,locate,tell,who:*
AgeOfHeroes:128.76.165.163:2347:channel,finger,tell,who,mail,locate:*
Aldebaran:85.214.77.105:4246:channel,finger,locate,tell,who,mail,man:*
Avalon:85.10.205.77:4246:channel,finger,locate,man,tell,who,mail,gopher:*
Beutelland:128.130.95.62:4246:tell,who,channel,finger,locate,mail:*
DeeperTrouble:172.104.251.185:8889:channel,finger,locate,tell,who:*
DevDune:138.197.134.82:6791:*:*
DragonfireII:97.95.18.87:2000:channel,finger,tell,who,mail,locate:*
Efferdland:152.53.16.20:4246:channel,finger,tell,who,mail,www,htmlwho,locate:*
EotL:38.86.32.239:4246:channel,finger,locate,man,tell,who:*
FinalFrontier:78.46.121.106:7610:channel,finger,tell,locate,who,mail,man,update,webquery:*
Kerovnia:151.198.54.56:1985:channel,finger,encoding,locate,mail,man,newsgroup,tell,who:*
Magicmud:152.53.16.20:3002:channel,finger,tell,who,mail,www,htmlwho,locate:*
MorgenGrauen:89.58.11.82:4246:channel,finger,tell,who,mail,www,htmlwho,locate:*
Nightfall:82.153.225.173:4246:channel,finger,encoding,locate,man,tell,who,mail,www:*
OuterSpace:159.69.87.242:3002:*:*
Realmsmud:68.57.196.242:4246:*:*
Seifenblase:217.11.52.247:4246:channel,finger,locate,man,tell,who,mail,gopher:*
SilberLand:77.237.49.230:4246:channel,finger,tell,who,mail,www,htmlwho,locate,man:*
Tamedhon:212.132.115.155:4246:channel,finger,tell,who,mail,www,htmlwho,locate:*
Tauros:34.221.136.93:5050:*:*
Theloria:178.254.12.82:3335:channel,finger,tell,locate,who,mail,newsgroup,portal:*
Tubmud:85.214.44.4:7683:channel,finger,locate,man,tell,who,www,mail:*
UNItopia:217.11.52.248:3335:channel,finger,tell,locate,who,mail,newsgroup:*
WL-Development:49.13.232.95:5757:channel,finger,tell,who,mail,locate,man:*
Wunderland:49.13.232.95:4246:channel,finger,tell,who,mail,locate,man:*

```
## History
The following paragraphs where written by the author of the protocol in May 1995

:::note
Dissapointed with the lack of a well implemented intermud system for Amylaar muds, I originally intended to simply port the CD intermud system. After examining the code for the CD system I became disenchanted by it's design and what I saw as limitations in it's protocol and implementation -- in particular it's limitations on the size of data transferred and it's lack of any form of reliability scheme.

Around this time Alvin@Sushi (aka. James Armstrong) began experimenting with his own UDP-based intermud system which included simple handshaking. I was involved in testing of this code and, inspired by this approach, began developing my own system based on the concepts conceived by Alvin.

Since it conception, the intermud package (often referred to as the "inetd" system) has become a fully fledged and functional intermud communication system. More recently, testing and development has been done on both Amylaar and CD based muds to ensure that the code is fully portable between the two systems. It also allows the CD intermud system to run alongside it to provide the greatest flexibility and compatability to mud admins.

Mark / Nostradamus@Zebedee.
:::
