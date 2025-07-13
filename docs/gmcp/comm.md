---
sidebar_label: comm
---
# The ``Comm`` package

Source: [Aardwolf](https://www.aardwolf.com/wiki/index.php/Clients/GMCP#aardmodules_comm)

## comm.channels

The 'comm.channels' set of tags contain channel data and include regular channels, auction/market, tell, say and yell. Unlike the old output tags, these all share the same type of message format and the channel name within the GMCP data is what changes: 
````json
comm.channel { 
    "chan": "gossip", 
    "msg": "You gossip 'Testing'", 
    "player": "Abelinc" }
````
- **chan**
  (*Mandatory*) Channel identifier
- **msg**
  (*Mandatory*) Message to display
- **player**
  (*Mandatory*) Player that sends the message


## comm.tick
````json
comm.tick { }
````

## comm.quest
Sends information when anything related to a quest happens. 

### Starting a quest
````json
comm.quest {
    "action": "start", 
    "targ": "a swamp ape", 
    "room": "Swamp Ape Enclosure", 
    "area": "Aardwolf Zoological Park", 
    "timer": 52 }
````

### Failing a quest
````json
comm.quest {
    "action": "fail", 
    "wait": 15 
}
````

### Completing a quest
````json
comm.quest { 
    "action": "comp", 
    "qp": 16, 
    "tierqp": 9, 
    "pracs": 0, 
    "hardcore": 0, 
    "opk": 0, 
    "trains": 0, 
    "tp": 0, 
    "lucky": 0, 
    "double": 0, 
    "daily": 1, 
    "totqp": 50, 
    "gold": 4831, 
    "completed": 111, 
    "wait": 30 
}
````

### Running out of time:
````json
comm.quest {"action": "timeout", "wait": 30 }
````

### Quest target killed:
````json
comm.quest {"action": "killed", "time": 52 }
````

### Quest time warning
````json
comm.quest {"action": "warning", "time": 5 }
````

### Can now quest
````json
comm.quest {"action": "ready" }
````

### Reset quest
````json
comm.quest {"action": "reset", "timer": 1 }
````

## comm.repop
Sent by the server when an area is repopulated

````json
comm.repop { "zone": "aylor" }
````
