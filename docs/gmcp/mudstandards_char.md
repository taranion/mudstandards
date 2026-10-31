---
sidebar_label: mudstd.char
---
# The ``mudstd.char`` package

## Attributes

### mudstd.char.attributes.definitions

```json
mudstd.char.attributes.definitions {
    {
    	"id": "str",
    	"label": "Strength",
    	"abbrev": "STR"
	},  
	{
        "id": "agi",
        "label": "Agility",
        "abbrev": "AGI"
    }
}
```

### mudstd.char.attributes.update

This command is sent from the server to the client to inform of the current state of the characters attributes. 

```json
mudstd.char.attributes.update {
    {
    	"id": "str",
    	"current": 8,
    	"max": 10
	},  {
        "id": "agi",
        "current": 7,
        "augmented": 9,  
        "max": 10
    }
}
```

- **id**

  (*Mandatory*) Refers to the attribute for which these stats are valid

- **current**

  (*Mandatory*) The current value of the attribute. Usually numerical

- **augmented**

- (*Optional*) The value improved by gear, spells or other effects

- **max**

  (*Optional*) The maximum value of the attribute - usually identical to the current value, unless some temporary effects raise or lower the attribute