---
sidebar_label: loci.menu
---
# The ``Loci.Menu`` package

A GMCP module to module to send server-defined menus to the client. <br/>
If client claims support, server will send menu definition update
    messages asynchronosly, as they become available, at the server's
    discretion.
Source: [LociTerm](https://www.last-outpost.com/LO/protocols/loci.menu.html)


## Loci.Menu.Get 
Request for server to send/resend all of the current menu definitions to
    the client.  May be sent at any time, or not at all.  Intended use is to
    indicate to server that all previously sent updates may have been lost due
    to a client reset, and need to be resent.  The server does not have to
    reply to this message, but it may respond with a ``Loci.Menu.Set``.
    
````json
"Loci.Menu.Get" { }
````

## Loci.Menu.Set
Message from server to client containing the complete menu theme redefinition.  May be sent at any time.
 
````json
"Loci.Menu.Set" { 
    name: "<menu name>",   
    label: "<menu label>"  
    menubox: { <menubox json> } 
    menubar: { <menubar json> } 
}
````
| Key     | Required | Value       | Description                                                  |
| ------- | -------- | ----------- | :----------------------------------------------------------- |
| name    | Yes      | String      | A valid identifier (see above) for the key to define     |
| label   | Yes      | String      | Name to appear in on-screen button                       |
| menubox | Useful   | JSON String |           |
| menubar | No       | JSON String |           |

 Intent is to send the complete menu redefinition
    early in the session and not change it after that. Server should avoid
    changing the menus "on the fly" in the middle of a game session, since the
    Set command performs a wholesale replacement of the current menuing
    system, and the player might be actively navigating some menu.  A more
    targeted update and replace for single items in the menu tree is deferrd to
    a future Loci.Menu release.

## Loci.Menu.Reset
Message from server to client indicating that the menu system should be
    returned to the user's saved preference, or to the default menu theme if no
    preference is saved.

````json
"Loci.Menu.Reset" { }
````

## Loci.Menu.Open
Message sent from server to client indicating that a menu UI element should be opened.
````json
"Loci.Menu.Open" <json string, id of menu element to open>
````
## Loci.Menu.Close
Message sent from server to client indicating that a menu UI element should be closed.
````json
"Loci.Menu.Close" <json string, id of menu element to open>
````

## Format of menu definition JSON:

This is the rough structure of a loci menu definition.  See the files in
client/src/menu/ for specific examples.

````
Menu Object:
    name "name of menu, lowercase, no spaces"
    label "display name of menu"
    menubox {}
        width # width of the box in buttons
        height # height of the box in buttons
        buttons []
            name "name of button"
            send "text to send to game, possibly through nerfbar, should end with \n"
            svgid "depreciated do not use"
            svgclass "depreciated do not use"
            menubar "id of menubar item to open"
            color "a css color"
            background "a css color"
            text "text of the button"
            img "href source of button's icon image"

    menubar {}
        id "id of this menubar"
        item []
            label "display name of item"
            open "id of menubar to open"
            prompt "partial text to send to game, possibly through nerfbar, must not end in \n"
            send "text to send to game, possibly through nerfbar, should end with \n"
            id "hotkey id (special use for hotkey identification.)"
            hotkey "keystrokes to send immediately to terminal if no 'send' exists, skipping nerfbar"
            color "a css color"
            background "a css color"
            direct "non-hotkey, value of keystrokes to send immediately to terminal, skipping nerfbar"
````
**Notes:**

The menu definition will automatically include a button in the upper right
corner of the menubox to allow opening the client's system menu. This can NOT
be overridden.  The user is always expected to be able to access a local
client control menu through that button.

Custom menu names SHOULD all begin with the "menu_" prefix. They MUST NOT begin
with "sys_", and that prefix will be ignored.

Built in system menus should start with the "sys_" prefix.  They are provided
by the client locally. These menus may be dynamic and re-built programatically
(ex: sys_hotkey, sys_wordstack), or they may open larger UI dialog windows (ex:
sys_about, sys_settings).  Menus in the "sys_" category may be opened from a
custom menu definition with an "open": "sys_&lt;whatever&gt;" directive.  HOWEVER, it
is understood that sys_ menus are client implmentation specific, and may not be
available in all clients that support the Loci.Menu protocol!  

For maximum compatibility, a Loci.Menu.Set defintion should try and define all
of the sub menus that it needs as menu_ type menus, and not rely on local sys_
menus at all.

It is also recommended that a client implmenting the Loci.Hotkey protocol
SHOULD make a list of hotkeys available under the sys_hotkey menu definition.

Some sys_menu definitions that are available in LociTerm:

"sys_about"
"sys_about_which"
"sys_client"
"sys_client_settings"
"sys_connect_direct"
"sys_disclaimer"
"sys_disconnect"
"sys_filters"
"sys_game_about"
"sys_game_select"
"sys_hotkey"
"sys_loginbox"
"sys_pronoun"
"sys_settings"
"sys_what"
"sys_who"
"sys_wordstack"

(A command to list the available sys_ menu items to the server is deferred to a
future Loci.Menu release.)