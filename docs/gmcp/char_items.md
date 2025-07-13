---
sidebar_label: char.items
---
# The ``Char.Items`` package

## Char.Items.Contents 

Sent by the client to the server.
Request for the server to send the list of items located inside another item.
````json
Char.Item.Contents <vnum>
````
- **vnum**
  (*Mandatory*) Number identifying the item

## Char.Items.Inv
Request for the server to send the list of items in player's inventory.
````json
Char.Items.Inv ""
````
## Char.Items.List 

Sent by the server. Contains a list of items at a specified location (room, inv, held container)
````json
Char.Items.List { 
    "location": "inv", 
    "items": [ 
        { 
            "id": "12807", 
            "name": "a personal journal", 
            "icon": "scroll", 
            "attrib": "l" 
        }, { 
            "id": "303060", 
            "name": "a gold nugget", 
            "icon": "commodity" 
        } 
    ] 
}
````
- **location**
  (*Mandatory*) Location value is a string, "inv", "room", or "repNUMBER" - the last one is container identification
- **items**
  (*Mandatory*) Items value is an array, whose each item is an object with keys "id", "name" and optionally "attrib"
- **id**
  (*Mandatory*) a number identifying the item, name is a string containing a short player-visible item description
- **name**
  (*Mandatory*) Guess what
- **icon**
  (*Mandatory*) the image the item is associated with in the client.
- **attrib**
  (*Optional*) Attrib is a string consisting of characters describing item properties:
    "w" = worn,
    "W" = wearable but not worn
    "l" = wielded
    "g" = groupable
    "c" = container
    "t" = takeable
    "m" = monster
    "d" = dead monster
    "x" = should not be targeted (guards, ...)



## Char.Items.Room 
Sent by the client to refresh the items in a room.
````json
Char.Items.Room ""
````

## Char.Items.Add

    Informs the client about an item being added to the specified location
    Message body is an object with keys "location" and "item"
    Location is same as with List, item is an object with the same structure as one item from the items array of List
````json
Char.Items.Add { 
    "location": "room", 
    "item": { 
        "id": "239602", 
        "name": "an elegant white letter", 
        "icon": "container", 
        "attrib": "c" 
    } 
}
````

````json
Char.Items.Add { "location": "room", "item": { "id": "303060", "name": "a gold nugget", "icon": "commodity" } }
````

````json
Char.Items.Add { 
    "location": "inv", 
    "item": { 
        "id": "303060", 
        "name": "a gold nugget", 
        "icon": "commodity", 
        "attrib": "t" 
    } 
}
````

## Char.Items.Remove

    Informs the client about an item being removed from the location
    Message body is an object with keys "location" and "item"
    Location is same as with List, item is an integer value identifying the item
````json
Char.Items.Remove { "location": "room", "item": { "id": "239602", "name": "an elegant white letter", "icon": "container", "attrib": "t" } }
````
````json
Char.Items.Remove { "location": "inv", "item": { "id": "303060", "name": "a gold nugget", "icon": "commodity" } }
````

## Char.Items.Update

    Informs the client about an item's attributes being changed - only sent for inventory items
    Message body syntax the same as with Add
````json
Char.Items.Update { "location": "inv", "item": { "id": "60572", "name": "an ornate steel rapier" } }
````

