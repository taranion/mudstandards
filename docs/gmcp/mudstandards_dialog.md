---
sidebar_label: mudstd.resources
---
# The ``mudstd.dialog`` package

## Motivation

This event package can be used for a server to send metainformation about NPCs the player is talking to and dialog choices

## Commands

Upon receiving knowledge that the client supports this package, the server should send the definitions event followed by the update event - either immediately or as soon as the player enters the game.

### mudstd.dialog.image

This event transmits the image of the current NPC speaker and is sent by the server when a dialog starts or the speaker within a dialog changes.

````json
mudstd.dialog.image {
    "url": "http://example.com:2000/npc/npc124.jpg",
    "title": "The Mayor"
}
````

- **url**
  (*Mandatory*) An HTTP(S) or [RFC 2397](https://datatracker.ietf.org/doc/html/rfc2397) URL that points to a small image (suggested 32x32 pixel) that represents the resource.
  
- **name**
  
  (*Optional*) The name to display for the speaker
  

### mudstd.dialog.topics

A list of suggested dialog choices to choose from.

```json
mudstd.dialog.topics {
    {
		"command": "1",
    	"label": "Who are you?"
	}, {
        "command": "2",
        "label": "Have you heard any rumours?"
    }, {
        "command": "3",
        "label": "Can you lend me some money?",
        "req": "Fast Talk 4",
        "isMet": false
    }, {
        "command": "4",
        "label": "Good bye",
        "exit": true
    }
}
```

* **command** (String)
  (*Mandatory*) The command to send to the MUD when this option is choosen.

* **label** (String)
  (*Mandatory*) The question or answer text. Not necessarily identical in the same words used to print when the option is selected, but should be in the same spirit.

* **emoji** (String)

  (*Optional*) A single glyph UTF-8 sequence

* **req** (String)
  (*Optional*) A human-readable requirement to be able to choose this option.

* **isMet** ("true"|"false")
  (*Optional*) If a `req` attribute was present this flag indicates if the requirement is met. A game can decide only to  omit the `req` and `isMet` attributes of the requirement is met.

* exit (`true`|`false`)
  (*Optional*) If `true` this option ends the dialog.
