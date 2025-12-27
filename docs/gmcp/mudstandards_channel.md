---
sidebar_label: ms.channel
---
# The ``ms.channel`` package

The purpose of this package is to send messages that are associated with specific communication channels a MUD server may provide.
All messages are encoded as UTF-8. Message content MUST be provided in a version without any control codes, but MAY additionally by provided with ANSI codes.
Channels can optionally have a assignment for a default color, but those can be overwridden using control ANSI codes in the messages.

## ms.channel.definitions

This command is sent once when the connection is established. It may be repeated if the channels available to the user change (e.g. by getting access to a new channel while playing).

### Example

````json
ms.channel.definitions {
    {
    	"id": "gtell",
    	"label": "Group Tell",
      	"color": "FF0000"
	},  
    {
        "id": "newbie",
        "label": "Newbie",
        "color": "00FFFF",
        "colorANSI": 3
    }
}
````
### Parameter

| Property                 | Type    | Required | Description                                                                          |
| ------------------------ | ------- | ----- | ----------------------------------------------------------------------------------- |
| [id](#id )               | string  | **Mandatory** |  A unique internal identifier for the channel - not intended to be displayed |
| [label](#label )         | string  | **Mandatory** |  The display name (e.g. for tabs, filters ..) of the channel                 |
| [color](#color )         | string  | **Optional** |  A RGB value for the channel color                                            |
| [colorANSI](#colorANSI ) | integer | **Optional** |  An ANSI color code from 0..15                                                 |

## ms.channel.event

Sent from the server when something was written on a channel

### Example
```json
ms.channel.event {
    {
    	"id": "gsay",
    	"sender": "Taranion",
    	"msg": "Taranion says to the group: Hello all!"
	}
}
```
### Parameter

| Property   | Type    | Required | Description                                                                          |
| ---------- | ------- | ----- | ----------------------------------------------------------------------------------- |
| id         | string  | **Mandatory** |  The channel identifier |
| sender     | string  | **Optional**  |  The name of the sender. UTF-8, no color codes. Can be omitted for system messages.     |
| msg        | string  | **Mandatory** |  The message content. UTF-8, no color codes              |
| senderANSI | string  | **Optional**  |  The name of the sender. UTF-8, may contain ANSI codes  |
| msgANSI    | string  | **Optional**  |  The message content. UTF-8, may contain ANSI codes      |

