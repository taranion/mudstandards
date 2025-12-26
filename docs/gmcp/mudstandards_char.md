---
sidebar_label: mudstandards.char
---
# The ``mudstandards.char`` package

## Resources

:::note
This duplicates the ``mudstandards.resources`` package, but places it under the ``mudstandards.char`` namespace to group all character related features together.
I am not sure yet which one is the better approach.
:::

The term "resources" refers to fast changing energies like Health or Mana.

### mudstandards.char.resources.definitions

Sent by the server to inform the client of all resources tracked on the MUD. The server may send more resources as those that apply to the character, e.g. because class specific resources exist.

````json
mudstandards.char.resources.definitions {
    {
    	"id": "hp",
    	"label": "Hit Points",
    	"abbrev": "HP",
    	"color": "FF0000"
	},  {
        "id": "mana",
        "label": "Mana",
        "abbrev": "Ma",
        "color": "0000FF"
    }
}
````

- **id**
  (*Mandatory*) This identifier is used to refer to this resource when sending updates. It should consist of ASCII letters only.
- **label**
  (*Mandatory*) A human readable name
- **abbrev**
  (*Optional*, but recommended) A 2 or 3 letter name for this resource, that can be used in tables, healthbars ...
- **color**
  (*Optional*) Hexadecimal RGB code of a color to use for this resource

### mudstandards.char.resources.update

This command is sent from the server to the client to inform of the current state of the characters resources. Servers may send this in a fixed interval or only when resources are changing

```json
mudstandards.char.resources.update {
    {
    	"id": "hp",
    	"current": "15",
    	"max": "20",
    	"tempmax": "24"
	},  {
        "id": "mana",
        "current": "16",
        "max": "22",
        "blocked": "4"
    }
}
```

- **id**

  (*Mandatory*) Refers to the resource for which these stats are valid

- **current**

  (*Mandatory*) The current value of the resource. Usually numerical

- **max**

  (Mandatory) The maximum value of the resource

- **tempmax**

  (*Optional*) If the maximum value is temporary modified , this is the current maximum. 

- **blocked**

  (*Optional*) Some game systems assign/block fractions of a resource while sustaining an effect. This value can be used to describe this.

## Attributes

### mudstandards.char.attributes.definitions

```json
mudstandards.char.attributes.definitions {
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

### mudstandards.char.attributes.update

This command is sent from the server to the client to inform of the current state of the characters attributes. 

```json
mudstandards.char.attributes.update {
    {
    	"id": "str",
    	"current": "8",
    	"max": "10"
	},  {
        "id": "agi",
        "current": "7",
        "max": "10"
    }
}
```

- **id**

  (*Mandatory*) Refers to the attribute for which these stats are valid

- **current**

  (*Mandatory*) The current value of the attribute. Usually numerical

- **max**

  (*Optional*) The maximum value of the attribute - usually identical to the current value, unless some temporary effects raise or lower the attribute