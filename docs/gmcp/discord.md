---
sidebar_label: external.discord
---
# The ``External.Discord`` package

This package has been defined by Mudlet and IRE.

Source: [Mudlet Wiki](https://wiki.mudlet.org/w/Standards:Discord_GMCP)

## External.Discord.Hello
Sent by the client to announce the players Discord information from the MUD and receive an official server invite URL (if any).

The server replies with ``External.Discord.Info``
````json
External.Discord.Hello { 
    user: "person#1234", 
    private: true 
}
````
- **user**
  (*Mandatory*) Discord user name
- **private**
  (*Optional*) The ‘private’ field exists for the case that a game decides to provide a directory, but the user still wishes to provide their username for bot integrations or other purposes. This MUST be respected by the game.

## External.Discord.Info

Sent as a reply to External.Discord.Hello

````json
Client.Map {
  inviteurl: "https://discord.gg/#####", 
  applicationid: "…" 
}
````
- **inviteurl**
    (*Optional*) If the server has its own Discord server, this is the Invite url
- **applicationid** 
  (*Optional*) a custom application id for the game to use instead of the client’s. This allows the use of custom icons, a custom game name, etc. It is a long number held in a quoted string.
  
## External.Discord.Get
Retrieves an External.Discord.Status from the server.
````json
external.discord.get 
````

## External.Discord.Status
````json
external.discord.status { 
    smallimage: ["iconname", "iconname2", "iconname3"], 
    smallimagetext: "Icon hover text", 
    details: "Details String", 
    state: "State String", 
    partysize: 0, 
    partymax: 10, 
    game: "Achaea", 
    starttime: "timestamp for start" 
}
````
- **iconname**
  should be an array of strings to allow for multiple options, so as not to lock multiple clients into the same iconset or icon names. Clients should use the first icon named which is available. All icon names MUST be sent in lowercase.
- **details**
  (*Mandatory*) will very often be disused by the client in favour of "Playing \<Game Name\>"
- **starttime**
  Unix timestamp (seconds since epoch).
- **endtime**
  UNix timestamp. If given, a countdown will be displayed