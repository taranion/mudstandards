---
sidebar_label: mudstd.game
---
# The ``mudstd.game`` package

## Motivation

This event package can be used for a server to send metainformation about either the game world or from the game mode.

## Commands

Upon receiving knowledge that the client supports this package, the server should send the definitions event followed by the update event - either immediately or as soon as the player enters the game.

### mudstd.game.mode

Sent by the server to inform the client of a high level mode change where user input might be interpreted differently. An example could be entering or leaving a combat or travelling in some kind of overworld mode.

For clients that support more packages they could adjust the layout of UI components, e.g. hiding components only used in combat while shopping - or provide a distraction free UI during online creation.

````json
mudstd.game.mode {
   "id": "exploring"
}

mudstd.game.info {
    "id": "olc",
    "submode": "room"
}

mudstd.game.info {
    "id": "menu",
    "submode": "mudmail"
}
````

- **id**
  (*Mandatory*) High level game mode
  
  Valid values are
  
  | id                 | Description                                                  |
  | ------------------ | ------------------------------------------------------------ |
  | start_screen       | Game server greeting screen                                  |
  | menu               | Some kind of menu with a limited list of choices             |
  | character_creation | Well, guess what                                             |
  | exploring          | The default mode of walking around and inspecting things     |
  | combat             | Fighting something                                           |
  | dialogue           | Talking to an NPC                                            |
  | shop               | Either buying or selling stuff                               |
  | overland           | A more map-oriented view                                     |
  | olc                | Online creation                                              |
  | other              | Used for all modes not represented above<br />Please use `submode` to specify the details |
  
- **submode**
  
  (*Optional*) Up to the server 

