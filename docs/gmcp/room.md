---
sidebar_label: room
---
# The ``Room`` package

This is likely the most common package, since it is used to transmit room information from the server to the client.

Sources
* https://nexus.ironrealms.com/GMCP#Room
* https://www.aardwolf.com/wiki/index.php/Clients/GMCP#aardmodules_room

## Room.Info

Examples from Aardwolf
````json
room.info { 
    "num": 5922, 
    "name": "At the entrance of the park", 
    "zone": "zoo", 
    "terrain": "city", 
    "details": "", 
    "exits": { 
        "e": 5920, 
        "s": 5916, 
        "w": 12611 
    }, 
    "coord": { 
        "id": 0, 
        "x": 37, 
        "y": 19, 
        "cont": 0 
    } 
}
````
Non-mappable room in Aardwolf
````json
room.info { 
    "num": -1, 
    "name": "Emerald Clan Room", 
    "zone": "emerald", 
    "terrain": "", 
    "details": "", 
    "exits": {}, 
    "coord": { "id": -1, "x": -1, "y": -1 } 
}
````
Example from Iron Realms 
````json
Room.Info {
    "num": 12345, 
    "name": "On a hill", 
    "area": "Barren hills", 
    "environment": "Hills", 
    "coords": "45,5,4,3", 
    "map": "www.imperian.com/itex/maps/clientmap.php?map=45&level=3 5 4", 
    "exits": { 
        "n": 12344, 
        "se": 12336 
    }, 
    "details": [ "shop", "bank" ] }
````

````json
room.info {
    "num":1331250207,
    "id":"tutorial zone#4",
    "name":"What.. How.. Who am I?",
    "zone":"tutorial zone",
    "desc":"Here you can choose a deity if you wish, it will make you lose half as much experience upon death, you can ^cLOOK MONA^N to learn bit more.",
    "terrain":"metal",
    "move":"normal",
    "details":"",
    "exits":{
        "N":1331250212,
        "S":1331250203},
    "idexits":{
        "N":"tutorial zone#9",
        "S":"tutorial zone#0"},
    "coord":{"id":0,"x":-1,"y":-1,"cont":0}
}
````

- **num**
  (*Optional*) The VNum/room identifier. Most MUDs always send this number, while others decided to hold this information back to prevent automapping.
- **name**
  (*Mandatory*) The room name
- **area** or **zone**
  The encompassing entity that contains this room
- **terrain**/**environment**
  (*Optional*) The type of terrain for this room. Might help coloring on auto-maps. The values itself vary from MUD to MUD
- **details**
  (*Optional*) An array of flags associated with this room.
- **map**
  (*Optional*) map information - URL pointing to a map image, followed by X and Y room (not pixel) coordinates on the map
- **coords**
  (*Optional*) room coordinates on the map, though the way it is formatted varies.
  IRE: (string of numbers separated by commas - area,X,Y,X,building, building is optional
  Aardwolf: Same like IRE, but formatted as an JSON object
- **exits**
  (*Mandatory*, though sometimes empty)object containing exits, each key is a direction and each value is the number identifying the target room
- **idexits**
  (*Optional*) Like *exits* but points to string identifier for rooms

## Room.WrongDir
Sent if the player attempts to move in a non-existant direction using the standard movement commands.
Upon receiving this message, the client can safely assume that the specified direction does not lead anywhere at this time.
````json
Room.WrongDir "ne"
````

## Room.Players
Object containing player details, each key is the short name of the player and each value is the full name including titles for the player
````json
Room.Players [
    { 
        "name": "Tecton", 
        "fullname": "Tecton the Terraformer" 
    }, {
        "name": "Cardan", 
        "fullname": "Cardan, the Curious" 
    }
]
````
## Room.AddPlayer
Message body has the same object structure as Room.Players except that it only contains the one player being added to the room.
````json
Room.AddPlayer [
    { 
        "name": "Cardan", 
        "fullname": "Cardan, the Curious" 
    }
]
````
## Room.RemovePlayer
Message body has the same object structure as Room.Players except that it only contains the one player being removed from the room.
````json
Room.RemovePlayer "Cardan"
````