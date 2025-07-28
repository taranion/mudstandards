---
sidebar_label: MUSHLink
---
# MushLink

MushLink is primarily used on MUSHes. It uses a TCP connection in order to connect a MUD to the central server. In contrast to the other interMUD networks so far listed, MushLink's server  actually connects to the MUD instead of having the MUD connect to the  server. This connection consists of a bot that logs in as a special  user. This user uses special commands in order to both display and  receive messages from the MUD.

### Services supported by MushLink

- **channels** for chatting
- **mail** for sending mail to users on other MUDs
- **mud list** lets a user retrieve the list of MUDs on the network
- **tell** send a message to a user on another MUD
- **who** see who's online on another MUD

### Software

MushLink requires no special code to be used on a MUSH. However, a  MUSH needs to support the standard MUSH commands in order for the  MushLink bot to work. For MUSHs that do not support the standard MUSH  commands, it is possible to emulate them.

The MushLink bot itself can be downloaded from PennMUSH web site.