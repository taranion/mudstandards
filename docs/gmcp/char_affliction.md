---
sidebar_label: char.affliction
---
# The ``Char.Affliction`` package

All messages in this package are sent by the server.

## Char.Affliction.List

(Iron Realms) Sends an array of current character afflictions. 
````json
Char.Afflictions.List [ 
    { 
        "name": "weariness", 
        "cure": "eat kelp", 
        "desc": "Decreases cutting and blunt damage that you inflict by 30%." 
    }, 
    { 
        "name": "asthma", 
        "cure": "eat kelp", 
        "desc": "Makes you unable to smoke pipes." 
    }, 
    { 
        "name": "slow herbs", 
        "cure": "apply epidermal", 
        "desc": "Increases the time needed to regain herb balance by 1.25 seconds." 
    }, 
    { 
        "name": "nausea", 
        "cure": "eat nightshade", 
        "desc": "Causes periodic vomiting, which does damage and increases hunger." 
    } 
]
````
- **name**
  (*Mandatory*) The name of the affliction
- **cure**
  (*Mandatory*) The basic cure of the affliction. This is used for links to cure even though other options may exist to cure the affliction.
- **desc**
  (*Mandatory*) A description of what this affliction does.

## Char.Afflictions.Add 

Add an affliction to the character.

````json
Char.Afflictions.Add { 
    "name": "asthma", 
    "cure": "eat kelp", 
    "desc": "Makes you unable to smoke pipes." 
}
````

## Char.Afflictions.Remove 

Removes an affliction from the character.

````json
Char.Afflictions.Remove [ "asthma" ]
````
