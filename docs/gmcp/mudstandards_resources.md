---
sidebar_label: mudstandards.resources
---
# The ``mudstandards.resources`` package

## Resources

:::note
This duplicates parts of the ``mudstandards.char`` package, but places it under the ``mudstandards.resources`` namespace to allow usage for non-character related resources as well.
A decision which one is the better approach is pending.
:::

The term "resources" refers to fast changing energies like Health or Mana.

### mudstandards.resources.definitions

Sent by the server to inform the client of all resources tracked on the MUD. The server may send more resources as those that apply to the character, e.g. because class specific resources exist.

````json
mudstandards.resources.definitions {
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

### mudstandards.resources.update

This command is sent from the server to the client to inform of the current state of the characters resources. Servers may send this in a fixed interval or only when resources are changing

```json
mudstandards.resources.update {
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

