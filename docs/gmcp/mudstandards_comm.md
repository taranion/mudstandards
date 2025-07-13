---
sidebar_label: mudstandards.comm
---
# The ``mudstandards.comm`` package

## Channels



### mudstandards.comm.channel.definitions



````json
mudstandards.char.channel.definitions {
    {
    	"id": "gtell",
    	"label": "Group Tell",
      	"color": "FF0000"
	},  {
        "id": "newbie",
        "label": "Newbie",
        "color": "00FFFF"
    }
}
````

- **id**
  (*Mandatory*) This identifier is used to refer to this channel when sending messages. It should consist of ASCII letters only.
- **label**
  (*Mandatory*) A human readable name
- **color**
  (*Optional*) Hexadecimal RGB code of a color to use for messages in this channel

### mudstandards.comm.channel.event

Sent from the server when something was written on a channel

```json
mudstandards.comm.channel.event {
    {
    	"id": "gsay",
    	"sender": "Taranion",
    	"msg": "Taranion says to the group: Hello all!"
	}
}
```

- **id**

  (*Mandatory*) Channel the message was sent on

- **sender**

  (*Mandatory*) The player who sent the message

- **msg**

  (Mandatory) The message to print

  


