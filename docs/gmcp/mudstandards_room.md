---
sidebar_label: mudstandards.room
---
# The ``mudstandards.room`` package

### mudstandards.room.terrain

:::warning
This is a proposal and request for comments only. 
:::

````json
mudstandards.room.terrain {
    {
        "id": "city",
        "label": "City",
        "color": "C0C0C0",
        "tile_url": "data:image/jpeg;base64,/9j/4AAQSkZJRgABAgAAZABkAAD"
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
  (*Optional*) An HTTP(S) or [RFC 2397](https://datatracker.ietf.org/doc/html/rfc2397) URL that points to a small image (suggested 32x32 pixel) that represents the terrain.
- **color**
  (*Optional*) Hexadecimal RGB code of a color to use for this terrain


### mudstandards.room.info

````json
mudstandards.room.info {
    {
    	"id": "1/1/12",
    	"name": "On a hill",
    	"description": "The view from this hill is spectacular ... at least that is what you will tell anyone if asked.",
      	"terrain": "forest",
      	"exits": {
            "E": {
                "id": "1/1/13",
                "inverse": "N",
            }
        }
	}
}
````

- **id**
  (Mandatory) An internal identifier that can be non-numeric. The identifier should be serverwide unique.
- **name**
  (*Mandatory*) The name of the room
- **description**
  (*Optional*) The full room description
- **terrain**
  (*Optional*) A reference to a terrain definition. See `mudstandards.room.terrain`
- **exits**
  (*Mandatory*) A map of exists, identified by the direction. Each entry consists of a map of exit object. The abbreviated direction (N,S,E,W,U,D,NE,NW,SE,SW) serves as a key. Valid attributes of an exit are:
  - **id** (*Mandatory*) 
    Identifier (possible non-numeric) of the target room.
  - **inverse** (*Optional*)
    Direction in which you can return from the target room back to this room. Useful for mappers in maps where exit directions are not simple inverse.
  

### mudstandards.room.entities

This command sends a list of NPCs/mobiles/players and items that are in the room (or near surrounding) to interact with.

````json
mudstandards.room.entities {
    {
         "name": "<display name for the entity",
         "type": "[mobile|item|player]",
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

- **type**
  (Mandatory) What kind of entity this is. Valid values are `mobile`, `item` and `player`.
  
- **name**
  (*Mandatory*) A short display name for the entity. It should not contain ANSI color codes.
  
- **nameANSI**
  (*Optional*) A short display name for the entity. It may contain ANSI color codes.
  
- **icon_url**
  (*Optional*) Either an HTTP(S) or [RFC 2397](https://datatracker.ietf.org/doc/html/rfc2397) URL that points to a small image (suggested 32x32 pixel) that represents the entity.
  
- **actions**
  (*Optional*) A short list of actions the client may offer to the user to interact with this entity. If the action is selected/clicked, the given command string is sent to the MUD as if the user had typed it. Valid attributes of an action are:
  
  - **name** (*Mandatory*)
    A very short (suggestion: one word olny) display name, e.g. to use for a button below the name, of the action
  
  - **command** (*Mandatory*)
    A command string that the client sends when the action is selected by the user
  
  - **emoji** (*Optional*)
    A emoji that can be prepended to the *name*. Gets an extra attribute, because some clients may not be able to use Emojis.
  
  - **color** (Optional)
    Not a color code, but a way to express potentially aggressive actions (like "kill", "steal"). How exactly a client renders this, is up to the client. Defaults to "normal".
  
    
