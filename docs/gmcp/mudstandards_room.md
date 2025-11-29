---
sidebar_label: mudstandards.room
---
# The ``mudstandards.room`` package

### mudstandards.room.terrain.definitions

````json
mudstandards.room.terrain.definitions {
    {
        "id": "city",
        "label": "City",
        "color": "C0C0C0",
        "tile_url": "data:ABCDEF1234567890"
    },  {
        "id": "forest",
        "label": "Forest",
        "color": "0000FF",
        "tile_url": "htts://myserver/tiles/forest.jpg"
    }
}
````

- **id**
  (*Mandatory*) Internal name to refer to from Room.Info. It should consist of ASCII letters only.
- **label**
  (*Mandatory*) A human readable name
- **tile_url**
  (*Optional*) An URL that points to a small image (suggested 32x32 pixel) that represents the terrain.
- **color**
  (*Optional*) Hexadecimal RGB code of a color to use for this terrain


### mudstandards.room.info

````json
mudstandards.room.info {
    {
    	"id": "1/1/12",
    	"num": "10112",
    	"name": "On a hill",
    	"description": "The view from this hill is spectacular ... at least that is what you will tell anyone if asked.",
      	"terrain": "forest",
      	"exits": {
            "e": {
                "id": "1/1/13",
                "num": "10113",
            }
        }
	}
}
````

- **id**
  (*Optional*) An internal identifier that can be non-numeric. The identifier should be serverwide unique.
- **num**
  (*Optional*) An internal identifier that is numeric, like a room-number. The number should be serverwide unique.
- **name**
  (*Mandatory*) The name of the room
- **description**
  (*Optional*) The full room description
- **terrain**
  (*Optional*) A reference to a terrain definition
- **exits**
  (*Mandatory*) A map of exists, identified by the direction. Each entry consists

### mudstandards.room.entities
This command sends a list of NPCs/mobiles and items that are in the room (or near surrounding) to interact with.

````json
mudstandards.room.entities {
    {
         "name": "<display name for the entity",
         "type": "[mobile|item]",
         "icon_url": "optional url to a small 32x32 pixel image",
         "actions": [
            {
                "name": "<display name of the command>",
                "command": "<string to send>",
                "emoji": "<optional emoji to prepend>",
                "color": "[normal|danger]"
            }
        ]
    }
}
````
