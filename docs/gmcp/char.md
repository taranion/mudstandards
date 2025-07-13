---
sidebar_label: char
---
# The ``Char`` package

## Char.Base

The 'char.base' set of data contains are items that will rarely change such as your class, subclass, race, clan, name. The char.base data is sent at login for scripts that want to capture this 'header' information and then not sent again unless one of these items changes. 

````json
char.base {
    "name": "Lasher",
    "class": "Warrior",
    "subclass": "Soldier",
    "race": "Elf",
    "clan": "wolf",
    "pretitle": "Testing",
    "perlevel": 1000
}
````

> Attention: The keys used in this command highly depend on the individual server.

## Char.Login

This message is sent by the client to log into a server

````json
Char.Login {
    "name": "somename",
    "password": "somepassword"
}
````

## Char.Name

This message is sent by the server

````json
Char.Name {
    "name": "Olad",
    "fullname": "Neophyte Olad"
}
````

## Char.Vitals

Sent by the server to inform the client of health, mana and moves information.
> Attention: The keys used in this command highly depend on the individual server.

Example from Aardwolf
````json
char.vitals { 
    "hp": 100000, 
    "mana": 90000, 
    "moves": 41599 
}
````

Example from Starmourn
````json
Char.Vitals { 
    "hp": "350", 
    "maxhp": "350", 
    "mp": "350", 
    "maxmp": "350", 
    "ep": "600", 
    "maxep": "600", 
    "wp": "600", 
    "maxwp": "600", 
    "nl": "0", 
    "bal": "1", 
    "eq": "1", 
    "vote": "1", 
    "string": "H:350/350 M:350/350 E:600/600 W:600/600 NL:0/100 ", 
    "charstats": [ "Bleed: 0", "Rage: 0" ] 
}
````

**Noteworthy**
- Some servers send (e.g. IRE serves) vitals as strings containing numbers, while others use integer datatypes.
- There is no common ground for a minimum set of vitals

## char.stats
Source: [Aardwolf](https://www.aardwolf.com/wiki/index.php/Clients/GMCP)
Sent by server to inform the client of the players stats.
**Stats are game dependent and vary from server to server.**

```json
char.stats { "str": 251, "int": 250, "wis": 250, "dex": 250, "con": 250, "luck": 250, "hr": 2298, "dr": 207, "saves": 13 }
```

## char.maxstats
Source: [Aardwolf](https://www.aardwolf.com/wiki/index.php/Clients/GMCP)
Sent by server. Contains max values for stats. In a separate group because these change far less often:
**Stats are game dependent and vary from server to server.**

```json
char.maxstats { "maxhp": 50099, "maxmana": 50029, "maxmoves": 41629, "maxstr": 51, "maxint": 134, "maxwis": 50, maxdex": 183, "maxcon": 99, "maxluck": 200 }
```

## char.statusvars

Sent by server after a successful login or after the module is enabled.
Contains a list of character variables (level, race, etc) and their human readable meaning.

```json
Char.StatusVars { "level": "Level", "race": "Race", "guild": "Guild" }
```

The variables are referenced in the ``char.status`` command

## char.status
```json
char.status { "level": "58", "city": "Antioch" }
```
**Example from Aardwolf**
```json
char.status { "level": 210, "tnl": 1000, "hunger": 70, "thirst": 70, "align": 1867, "state": 3,  "pos": "Standing" , "enemy": "an owl", "enemypct": 93 }
```

## char.worth
These fields are related to achievements or 'worth' in the game and cover qp, tp, etc:
 
**Example from Aardwolf**
```json
char.worth { "gold": 23128310661, "bank": 750000, "qp": 5052186, "tp": 10930, "trains": 6, "pracs": 14, "qpearned": 12345678 }
```
