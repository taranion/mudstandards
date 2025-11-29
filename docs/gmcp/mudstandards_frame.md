---
sidebar_label: mudstandards.frame
---
# The ``mudstandards.frame`` package

This GMCP package intends to let a MUD server open/close "*frames*" at the client and either direct output to it or open a webview with a given URL.

## Definitions

### Frame types
A frame can be one of the following types
* **External**
  An external frame is a window outside the main area of the client. Opening an external area should not change the size of the main client area.
* **Docked**
  A docked frame is a window that is docked to a border of the main area and reduces the effective size of the main area.
* **Floating**
  This is a window that is floating internally above the main area.
* **Child**
  This is a nested area inside another that is split off from the parent
* **Tab**
  This is a sibling area to another area. This area is to be displayed as an alternative to the sibling area, visualized as a Tab

### Frame content types
There are three kinds of window content an area can have
* **Terminal**
  Comparable to the main area of the client, this area can display ANSI text content
* **WebView**
  This area can be given an URL of a HTML+Javascript page. The client must expose a Javascript object to send and subscribe to GMCP commands (see TODO)
* **Image**
  The only content of the area is a single image, which of course can be updated.  

### Negotiating support
It is likely that clients do not support all area and content types. Upon connecting to a server, the client should send the ``mudstandards.area.support`` command.

### a name="size"></a>The size object
````json 
{
    "width"  : <number>,
    "height" : <number>
}
````

### The decoration object
````json 
{
    "background": "<url>",
    "scrolling" : <value>  none, X, Y, both,
    "closeable" : <boolean>,
    "resizeable": <enum>   none, X, Y, both,
    "label" : <string>,
    "opacity": <0..100>
}
````

## Commands

### mudstandards.frame.support
Sent by the client to notify the server of its capabilities. Should be send unsolicited after establishing the connection or as a response to a ``mudstandards.window.query``

````json
mudstandards.frame.support {
    "type": ["external","docked"], 
    "content": ["terminal","image"]
}
````

| Property       | Type    | Required | Description                                                                          |
| -------------- | ------- | ----- | ----------------------------------------------------------------------------------- |
| type            | List of [external\|docked\|floating\|child\|tab] | **Mandatory**|  Supported area types|
| content     | List of [terminal\|webview\|image] | **Mandatory**|  Supported area content types |


### mudstandards.frame.open

This message opens a new frame/window/dock in the client

````json
mudstandards.frame.open {
    "type": ["external","docked"], 
    "content": ["terminal","image"]
}
````

| Property       | Type    | Required | Description                                                                          |
| -------------- | ------- | ----- | ----------------------------------------------------------------------------------- |
| id             | string           | **Mandatory**|  A unique identifier for the room.                                                                                                                                                 |
| label          | string           | **Optional**| The name to appear above the window. Without it, no space for a label should be reserved                                                                                                       |
| type            | [external\|docked\|floating\|child\|tab] | **Mandatory**|  How will this frame be handled: An external (moveable) window, a docked frame or a  child frame of another window or a tab, that can be displayed alternatively to the *parent*|
| parent       | string           | **Optional**|  For windows of type *child*, the parent frame they belong to. For windows of type *tab*, the reference frame to alternate with.                                                    |
| align       | [top\|bottom\|left\|right] | **Conditional**|  On which side of the reference window shall this window appear                                                                                                                  |
| sizeValue   | integer          | **Conditional**|  Size of the new window. Consult *sizeUnit* for meaning                                                                                                                          |
| sizeUnit    | [c\|px\|%] | **Optional**|  Is the size to be interpreted as characters(c), pixel(px) or percent(%)                                                                                                            |
| content     | [terminal\|webview] | **Optional**|  What type of content should be displayed in that frame? <br />\`content="terminal"\` is another terminal emulator, while \`content="webview"\` is an HTML webpage with Javascript. |
| url          | url              | **Conditional**|  In case of a `webview` content, this contains the URL to open                                                                                                                   |

### mudstandards.frame.close
Sent from the server to request the closing of a frame.

````json
mudstandards.window.close { 
    "id":   "topleft"
}
````

| Property       | Type    | Required | Description                                                                          |
| -------------- | ------- | ----- | ----------------------------------------------------------------------------------- |
| id         | string  | **Mandatory** |  The identifier of the area to close |


### mudstandards.frame.terminal

This commands writes ANSI content to a given window/frame of type `terminal`.

````json
mudstandards.frame.terminal { 
    "id":   "stats",
    "clear" :   true,
    "ansi" :    "\x1b[0;1;37mSTR:\X1b[0m 12"    
}
mudstandards.frame.terminal { 
    "id":   "channel",
    "ansi" :   "\x1b[0;1;37mFoo says, 'Bar!'\x1b[0m"}"    
}
````

| Property       | Type    | Required | Description                                                                          |
| -------------- | ------- | ----- | ----------------------------------------------------------------------------------- |
| id         | string  | **Mandatory** |  The identifier of the area to output the content |
| ansi           | string  | **Mandatory** |  The UTF-8 encoded content (with potential ANSI codes) to output      |
| clear          | boolean  | **Optional** |  If `true`, the window should be cleared before the output    |


### mudstandards.frame.image

This commands updates an image to a given window/frame of type `image`. The image can be given as an URL or as a Base64 encoded inline image.

````json
mudstandards.frame.image { 
    "id":   "topleft",
    "image" :   "base64:<base64data>"    
}
mudstandards.frame.image { 
    "id":   "topleft",
    "image" :   "http://myserver.com/portrait.png"    
}
````

| Property       | Type    | Required | Description                                                                          |
| -------------- | ------- | ----- | ----------------------------------------------------------------------------------- |
| id         | string  | **Mandatory** |  The identifier of the area to output the content |
| image          | string  | **Mandatory** |  URI - either an image irl or base 64 encoded data with a `base64` schema        |


## Events

These commands are sent by the client when the frame setup changed

### mudstandards.frame.opened
This event is sent when the client opens or reopens a frame. It is meant to provide the server with size information about the frame.
````json
mudstandards.frame.openend { 
    "id":   "topleft",
    "sizeChar" :  <size object for character width/height>,
    "sizePixel":  <size object for pixel width/height>    
}
````
| Property       | Type    | Required | Description                                                                          |
| -------------- | ------- | ----- | ----------------------------------------------------------------------------------- |
| id             | string  | **Mandatory** |  The identifier of the area that has been opened |
| sizeChar       | [size object](#size )  | **Mandatory** |  The size of the area in character width and height      |
| sizePixel      | [size object](#size )  | **Optional** |  The inner size for content measured in pixel. If the client does scaling, the effective size after scaling should be used. |

### mudstandards.frame.closed
This event is sent when the client closes a frame - either because a user did so or because the server requested closing the frame.
````json
mudstandards.frame.closed { 
    "id":   "topleft",
    "reason" :  ["system"|"user"]  
}
````
| Property       | Type    | Required | Description                                                                          |
| -------------- | ------- | ----- | ----------------------------------------------------------------------------------- |
| id             | string  | **Mandatory** |  The identifier of the area that has been closed |
| reason       | ["system"|"user"]  | **Optional** |  Inform why the closing happened - "user" means by user request   |

### mudstandards.frame.resized
This event is sent whenever the frame size changes that much that a new character width or height is available. It is advised that 
the client does not send this events during a resizing operation, but when the resizing is finished. 
````json
mudstandards.frame.resized { 
    "id":   "topleft",
    "sizeChar" :  <size object for character width/height>,
    "sizePixel":  <size object for pixel width/height>    
}
````
| Property       | Type    | Required | Description                                                                          |
| -------------- | ------- | ----- | ----------------------------------------------------------------------------------- |
| id             | string  | **Mandatory** |  The identifier of the area that has been resized |
| sizeChar       | size object  | **Mandatory** |  The size of the area in character width and height      |
| sizePixel      | size object  | **Optional** |  The inner size for content measured in pixel. If the client does scaling, the effective size after scaling should be used. |
