---
sidebar_label: mudstd.resources
---
# The ``mudstd.resources`` package

## Why?	

The goal of this package is to provide an enough information that a generic MUD client can display gauges for stats like health, mana, movement points ... This is often done already in the [`char`](char.md) package, but is not very well suitable for that case, because there is

* no common agreement which fields should be displayed
* no definition if values should be sent as strings or integers
* no way to suggest which colors to use for which resource
* express more information than a maximum and current value

## Commands

Upon receiving knowledge that the client supports this package, the server should send the definitions event followed by the update event - either immediately or as soon as the player enters the game.

### mudstd.resources.definitions

Sent by the server to inform the client of all resources tracked on the MUD. The server may send more resources as those that apply to the character, e.g. because class specific resources exist.

````json
mudstd.resources.definitions {
    {
    	"id": "hp",
    	"label": "Hit Points",
    	"abbrev": "HP",
    	"color": "FF0000"
	},  {
        "id": "mana",
        "label": "Mana",
        "abbrev": "Ma",
        "color": "0000FF",
        "icon_url": "data:image/jpeg;base64,/9j/4AAQSkZJRgABAgAAZABkAAD"
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
  
- **colorANSI**
  (*Mandatory*)  A value from 0 to 15, referring to the ANSI colors

- **colorFaded**
  (*Optional*) A less vibrant, more reduced version of *color*, used to indicate resource points temporarily used or blocked

- **colorFadedANSI**
  (*Optional*) Same as *colorFaded* but expressed with a an ANSI color code

- **icon_url**

  (*Optional*) An HTTP(S) or [RFC 2397](https://datatracker.ietf.org/doc/html/rfc2397) URL that points to a small image (suggested 32x32 pixel) that represents the resource.

The server should sent the definitions in order of importance, assuming that a client can only show a limited number of resources at a time.

This event is sent either once per session or every time relevant resources change - e.g. in a combat other resources may be relevant than while exploring.

### mudstd.resources.update

This command is sent from the server to the client to inform of the current state of the characters resources. Servers may send this in a fixed interval or only when resources are changing

```json
mudstd.resources.update {
    {
    	"id": "hp",
    	"current": 15,
    	"max": 20,
        "exhausted": 0,
    	"tempmax": 24
	},  {
        "id": "mana",
        "current": 16,
        "max": 22,
        "blocked": 4
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

- **exhausted**
  *(Optional)* Some game systems work with some kind of short term resource loss, like "Stamina loss" that also reduces drain but is regenerated faster and falling to zero does not lead to death, but to unconciousness.
  
- **blocked**

  (*Optional*) Some game systems assign/block fractions of a resource while sustaining an effect. This value can be used to describe this.

If you are going to put this into a gauge, the formula is:

* max + tempmax = 100%, the full width of the gauge
* Gauge = current + exhausted + blocked

### mudstd.resources.request

Sent by the client to request the server to send a `mudstd.resources.definitions` and a `mudstd.resources.update`
