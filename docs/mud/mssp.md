---
sidebar_label: MSSP (Option 70)
---
# MUD Server Status Protocol (MSSP)

**Source**: [Mudhalla](https://mudhalla.net/tintin/protocols/mssp/)

MUD listings are often out dated and lack accurate information. Verifying that the one submitting a new MUD is a member of the MUD's administration can be quite tedious as well.
The MUD Server Status Protocol seeks to address these issues by providing a transparant protocol for MUD crawlers to gather detailed information about a MUD, including dynamic information like boot time and the current amount of online players. It also makes submitting a new mud entry very simple because it only requires a player or admin to fill in the hostname and port.
This document provides a technical description of the MSSP protocol.

## The MSSP Protocol

MSSP is implemented as a Telnet option RFC854, RFC855. The server and client negotiate the use of MSSP as they would any other telnet option. Once agreement has been reached on the use of the option, option sub-negotiation is used to send information about the server to the client.

### Server Commands
```
IAC WILL MSSP    indicates the server supports MSSP.
```
### Client Commands
```
IAC DO   MSSP    indicates the client supports MSSP.
IAC DONT MSSP    indicates the client doesn't support MSSP.
```
## Handshake
When a client connects to a server the server should send IAC WILL MSSP. The client should respond with either IAC DO MSSP or IAC DONT MSSP. If the server receives IAC DO MSSP it should respond with: IAC SB MSSP MSSP_VAR "variable" MSSP_VAL "value" MSSP_VAR "variable" MSSP_VAL "value" IAC SE.
The quote characters mean that the encased word is a string, the quotes themselves should not be send.

## MSSP definitions
```
MSSP 70

MSSP_VAR 1
MSSP_VAL 2
```

### Example MSSP handshake
```
server - IAC WILL MSSP
client - IAC DO MSSP
server - IAC SB MSSP MSSP_VAR "PLAYERS" MSSP_VAL "52" MSSP_VAR "UPTIME" MSSP_VAL "1234567890" IAC SE
```

### Variables and Values
For ease of parsing, variables and values cannot contain the MSSP_VAL, MSSP_VAR, IAC, or NUL byte. The value can be an empty string unless a numeric value is expected in which case the default value should be 0. If your Mud can't calculate one of the numeric values for the World variables you can use "-1" to indicate that the data is not available. If a list of responses is provided try to pick from the list, unless "Etc" is specified, which means it's open ended.
The same variable can be send more than once with different values, in which case the last reported value should be used as the default value. It is up to the crawler to decide how to exactly process multiple values, multiple values should be ordered from least to most relevant. It's also possible to attach several values to a single variable by using MSSP_VAL more than once, with the default value reported last. This would look as following:
```
IAC SB MSSP MSSP_VAR "PORT" MSSP_VAL "80" MSSP_VAL "23" MSSP_VAL "3000" MSSP_VAR "CREATED" MSSP_VAL "1996" IAC SE
```
The quote characters mean that the encased word is a string, the quotes themselves should not be send.
Variable names should exist of upper case letters and may contain spaces. As many programming languages have difficulties with variable names which contain spaces clients and crawlers can substitute spaces with underscores as the recommended solution.

## Official MSSP Variables

| Name         | Description                                    |
| ------------ | ---------------------------------------------- |
| **Required** |                                                |
| NAME         | Name of the MUD                                |
| PLAYERS      | Current number of logged in players            |
| UPTIME       | Unix time value of the startup time of the MUD |
| **Generic**  |                                                |
| CHARSET      | ASCII, BIG5, CP437, CP949, CP1251, EUC-KR, GB18030, ISO-8859-1, ISO-8859-2, KOI8-R, UTF-8.<br/>Name of the charset in use. You can report multiple charsets using the array format, the preferred / default charset last. See man charsets for reference. |
| CODEBASE     | Name of the codebase, eg Merc 2.1. You can report multiple codebases using the array format, make sure to report the current codebase last.|
| CONTACT      | Email address for contacting the MUD.          |
| CRAWL DELAY  | Preferred minimum number of hours between crawls. Send -1 to use the crawler's default.<br/>Recommended values are -1, 1, 5, 11, and 23.|
| CREATED      | Year the MUD was created.
| DISCORD      | URL to a Discord server, this should include the https:// prefix.
| HOSTNAME     | Current or new hostname.
| ICON         | URL to a square image in bmp, png, jpg, or gif format. The icon should be equal or larger than 64x64 pixels, with a filesize no larger than 256KB.
| IP           | Current or new IP address.
| IPV6         | Current or new IPv6 address.
| LANGUAGE     | English name of the language used, eg German or English
| LOCATION     | English short name of the country where the server is located, using ISO 3166.|
| MINIMUM AGE  | Current minimum age requirement, omit if not applicable.
| PORT         | Current or new port number. Can be used multiple times, most important port last.
| REFERRAL     | A list of other MSSP enabled MUDs for the crawler to check using the host port format and array notation. Adding referrals is important to make MSSP decentralized. Make sure to separate the host and port with a space rather than : because IPv6 addresses contain colons.
| SSL          | The port number for a SSL (Secure Socket Layer) encrypted connection.
| WEBSITE      | URL to MUD website, this should include the http:// or https:// prefix. |
| **Categorization** |    |
| FAMILY       | AberMUD, CoffeeMUD, DikuMUD, Evennia, LPMud, MajorMUD, MOO, Mordor, SocketMud,  TinyMUD, TinyMUCK, TinyMUSH, Custom.<br/>Report Custom unless it's a well established family.<br/>You can report multiple generic codebases using the array format, make sure to report the most distant codebase (aka the family) last.<br/>Check the MUD family tree for naming and capitalization.<br/>
| GENRE        | Adult, Fantasy, Historical, Horror, Modern, Mystery, None, Romance, Science Fiction, Spiritual
| GAMEPLAY     | Adventure, Educational, Hack and Slash, None, Player versus Player, Player versus Environment, Questing, Roleplaying, Simulation, Social, Strategy
| STATUS       | Alpha, Closed Beta, Open Beta, Live       |
| GAMESYSTEM   | D&D, d20 System, World of Darkness, Etc.<br/> Use Real Time, Tick Based, Turn Based, or Custom if using a custom gamesystems. Use None if not available.|
| INTERMUD     | AberChat, I3, IMC2, MudNet, Etc.<br/>Can be used multiple times if you support several protocols, most important protocol last. Leave empty or omit if no Intermud protocol is supported.
| SUBGENRE     | Alternate History, Anime, Cyberpunk, Detective, Discworld, Dragonlance, Christian Fiction, Classical Fantasy, Crime, Dark Fantasy, Epic Fantasy, Erotic, Exploration, Forgotten Realms, Frankenstein, Gothic, High Fantasy, Magical Realism, Medieval Fantasy, Multiverse, Paranormal, Post-Apocalyptic, Military Science Fiction, Mythology, Pulp, Star Wars, Steampunk, Suspense, Time Travel, Weird Fiction, World War II, Urban Fantasy, Etc.<br/>Use None if not applicable.
| **World**    |           | 
| AREAS        | Current number of areas.                  |
| HELPFILES    | Current number of help files.             |
| MOBILES      | Current number of unique mobiles.         |
| OBJECTS      | Current number of unique objects.         |
| ROOMS        | Current number of unique rooms, use 0 if roomless.|
| CLASSES      | Number of player classes, use 0 if classless.|
| LEVELS       | Number of player levels, use 0 if level-less.|
| RACES        | Number of player races, use 0 if raceless.|
| SKILLS       | Number of player skills, use 0 if skill-less.|
| **Protocols** |     | 
| ANSI         | Supports ANSI colors ? 1 or 0                 |
| UTF-8          | Supports UTF-8 ? 1 or 0
| VT100          | Supports VT100 interface ?  1 or 0
| XTERM 256 COLORS | Supports xterm 256 colors ?  1 or 0
| XTERM TRUE COLORS | Supports xterm 24 bit colors ? 1 or 0
| **Commercial** |    | 
| PAY TO PLAY  | Pay to play ? 1 or 0
| PAY FOR PERKS| Pay for perks ? 1 or 0
| **Hiring**   | | 
| HIRING BUILDERS | Game is hiring builders ? 1 or 0 |
| HIRING CODERS   | Game is hiring coders ? 1 or 0      |

