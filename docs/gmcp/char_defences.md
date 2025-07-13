---
sidebar_label: char.defences
---
# The ``Char.Defences`` package

All messages in this package are sent by the server.
Source: [Iron Realms](https://nexus.ironrealms.com/GMCP)

## Char.Defences.List

(Iron Realms) Send an array of all the defences a character currently has.

````json
Char.Defences.List [ 
    { 
        "name": "deaf", 
        "desc": "deaf" 
    }, { 
        "name": "blind", 
        "desc": "blind" 
    }, { 
        "name": "nightsight", 
        "desc": "nightsight" 
    } 
]
````
- **name**
  (*Mandatory*) The name of the defence
- **desc**
  (*Mandatory*) The description of the defence.

## Char.Defences.InfoList



````json
Char.Defences.InfoList [ 
    { 
        "name": "Jogging", 
        "desc": "Jogging around", 
        "category": "", 
        "important": "0", 
        "icon": "person-running", 
        "color": "white" }, 
    { "name": "Treading", "desc": "Treading carefully", "category": "", "important": "0", "icon": "shoe-prints", "color": "white" }, 
    { "name": "Lurking", "desc": "Lurking in shadows", "category": "", "important": "0", "icon": "eye-slash", "color": "white" }, 
    { "name": "Huddled", "desc": "Huddled in a ball", "category": "", "important": "0", "icon": "shield-exclamation", "color": "white" }, 
    { "name": "Overwatch", "desc": "Watching a direction", "category": "", "important": "0", "icon": "binoculars", "color": "white" }, 
    { "name": "Pacing", "desc": "Pacing the group you lead", "category": "", "important": "0", "icon": "people-group", "color": "white" }, 
    { "name": "Guarding", "desc": "Guarding another from assault", "category": "", "important": "0", "icon": "user-group-crown", "color": "white" }, 
    { "name": "Balancing", "desc": "Balancing skillfully", "category": "", "important": "0", "icon": "scale-balanced", "color": "red" }, 
    { "name": "Harden", "desc": "Altered skin", "category": "", "important": "0", "icon": "layer-group", "color": "red" }, 
    { "name": "Warmth", "desc": "Warmed skin", "category": "", "important": "0", "icon": "fire-flame", "color": "red" }, 
    { "name": "Firewreathe", "desc": "Wreathed in fire", "category": "", "important": "0", "icon": "flame-simple", "color": "red" } 
]
````
Parameters are mostly self-explanatory, but no documentation has been found

## Char.Defences.Add 

Sent when a defence is added to a character.

````json
Char.Defences.Add { 
    "name": "deaf", 
    "desc": "deaf" 
}
````

## Char.Defences.Remove 

Sent when a defence is removed from a character.

````json
Char.Defences.Remove [ "blind" ]
````
