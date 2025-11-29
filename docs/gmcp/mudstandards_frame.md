---
sidebar_label: mudstandards.frame
---
# The ``mudstandards.frame`` package

This GMCP package intends to let a MUD server open/close "*frames*" at the client and either direct output to it or open a webview with a given URL.

## Definitions

### <a name="ftype"></a>Frame types
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

### <a name="ctype"></a>Frame content types
There are three kinds of window content an area can have
* **Terminal**
  Comparable to the main area of the client, this area can display ANSI text content
* **WebView**
  This area can be given an URL of a HTML+Javascript page. The client must expose a Javascript object to send and subscribe to GMCP commands (see TODO)
* **Image**
  The only content of the area is a single image, which of course can be updated.  

### Negotiating support
It is likely that clients do not support all area and content types. Upon connecting to a server, the client should send the ``mudstandards.area.support`` command.

### <a name="size"></a>The size object
````json 
{
    "width"  : <number>,
    "height" : <number>
}
````
| Property   | Type     | Required | Description                        |
| ---------- | -------- | -------- | ---------------------------------|
| width      | integer | **Mandatory**|  Width of usable area |
| height     | integer | **Mandatory**|  Height of usable area |

### <a name="details"></a>The details object
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
| Property   | Type     | Required | Description                        |
| ---------- | -------- | -------- | ---------------------------------|
| background | string   | **Optional**|  Download URL of a background image |
| opacity    | integer  | **Optional**|  Opacity percentage of the frame background (incl. eventual image) |
| scrolling  | string  | **Optional**|  Will this frame scroll and if so, on which axis? <br/>**Options:** [none,X,Y,both] |
| resizeable | string  | **Optional**|  Can the user resize the frame and if so, on which axis? <br/>**Options:** [none,X,Y,both] |
| closeable  | boolean | **Optional**|  Can the user resize close the frame? |

## Commands

### mudstandards.frame.support
Sent by the client to notify the server of its capabilities. Should be send unsolicited after establishing the connection or as a response to a ``mudstandards.window.query``

````json
mudstandards.frame.support {
    "type": List of <frame type>, 
    "content": List of <content type>
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
    "id"  : <string>,
    "type": <frame type>, 
    "content": <content type>,
    "align": [top|bottom|left|right],
    "label": <string>,
    "parent": <string>,
    "sizeValue": <string>,
    "sizeUnit": <string>,
    "url" : <string>
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
| content     | [terminal\|[webview](#webview)] | **Optional**|  What type of content should be displayed in that frame? <br />\`content="terminal"\` is another terminal emulator, while \`content="webview"\` is an HTML webpage with Javascript. |
| url          | string              | **Conditional**|  In case of a `webview` content, this contains the URL to open                                                                                                                   |
| details     | [details object](#details )              | **Optional**|  Configuration details for the new frame. There is no guarantee that the client implements a specific detail.                                                                                                               |

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
mudstandards.frame.opened { 
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
| reason         | [system\|user] | **Optional** |  Inform why the closing happened - "user" means by user request   |

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
| sizeChar       | [size object](#size )  | **Mandatory** |  The size of the area in character width and height      |
| sizePixel      | [size object](#size )  | **Optional** |  The inner size for content measured in pixel. If the client does scaling, the effective size after scaling should be used. |

## <a name="webview"></a>Webview frames

Webviews are basically HTML pages with the possibility to send and receive data via GMCP.

Inside the webview HTML, JavaScript can access the host through this object:

``window.chrome.webview.hostObjects.client`` Note that this part: ``window.chrome.webview.hostObjects.`` is just where WebViews give access to host objects (host = BeipMU in this case). It's long because they want to avoid colliding with any user scripts.

As the webview runs in a separate process, all functions are asynchronous by default. To avoid needing to use 'await' on everything, do this first (it's done in the example page above):

``window.chrome.webview.hostObjects.options.defaultSyncProxy=true``

This is the client part of window.chrome.webview.hostObjects.client

Methods:
* **CloseWindow()**<br/>
  Close the WebView window. Useful if your webview is a popup to choose an item and you want it to close after doing your selection.
*  **Send(string text, bool process_aliases=false)**<br/>
  Send the given string as text over the connection.<br/>
  ``window.chrome.webview.hostObjects.client.send("page friend=\"Booo!");``<br/>
  <u>Parameters</u>
  * ``text`` The text to send
  * ``process_aliases`` Set to true to have the user's aliases processed when sending the text
* **SendGMCP(string cmd, string json)**<br/>
  Send the given command & json as a GMCP telnet message. 
  ````
  let gmcp=["BeipTest1 1", "BeipTest2 1"];
  window.chrome.webview.hostObjects.client.SendGMCP("Core.Supports.Add "+JSON.stringify(gmcp));
  ````
  <u>Parameters</u>
  * ``cmd`` The GMCP command name
  * ``json`` A string of the JSON

````html
<!DOCTYPE html>
<head>
    <title>Tile Map Viewer</title>
    <style>
        #mapCanvas { width: 100%; height: 100%; object-fit: contain; }
        body { display: flex; height: 100vh; background-color:black; margin: 0;}
    </style>
</head>
<body>
    <canvas id="mapCanvas"></canvas>

    <script>
        const client=window.chrome.webview.hostObjects.client;

        window.onload = async function() {
            client.SetOnConnect(OnConnect);
            client.SetOnGMCP("tilemap", OnGMCP);

            // Window was opened while we were already connected, so setup as though we just connected
            if(await client.IsConnected())
            OnConnect();
        }

        function OnConnect() {
          let gmcp=["tilemap 1"];
          client.SendGMCP("Core.Supports.Add", JSON.stringify(gmcp));
        }

        let map_info = null;
        let map_data;
        let tile_image;
        let tile_size;
        let map_size;

        function OnGMCP(package, json) {
            const data = JSON.parse(json);
            
            if (package === "tilemap.info") {
                map_info = data;
                document.title = map_info.title;
                tile_size = map_info.tile_size.split(',').map(Number);
                map_size = map_info.map_size.split(',').map(Number);
                tile_image = new Image();
                tile_image.src = map_info.tile_url;

                const canvas = document.getElementById("mapCanvas");
                canvas.width = tile_size[0] * map_size[0];
                canvas.height = tile_size[1] * map_size[1];
                return;
            }

            if (package === "tilemap.data" && map_info !== null) {
                switch (map_info.encoding) {
                    case "hex_4": map_data = data.match(/.{1}/g).map(hex => parseInt(hex, 16)); break;
                    case "hex_8": map_data = data.match(/.{2}/g).map(hex => parseInt(hex, 16)); break;
                    case "hex_12": map_data = data.match(/.{3}/g).map(hex => parseInt(hex, 16)); break;
                    case "base64_8": map_data = atob(data).split("").map(c => c.charCodeAt(0)); break;
                    default: console.error("Unknown encoding"); return;
                }

                if(tile_image.complete)
                    UpdateCanvas();
                else
                    tile_image.onload = UpdateCanvas;
            }
        }

        function UpdateCanvas() {
            const canvas = document.getElementById("mapCanvas");
            const context = canvas.getContext("2d");

            const bitmap_tile_width = tile_image.width / tile_size[0];
            for (let y = 0; y < map_size[1]; y++) {
                for (let x = 0; x < map_size[0]; x++) {
                    const tile_index = map_data[y * map_size[0] + x];
                    const sx = (tile_index % bitmap_tile_width) * tile_size[0];
                    const sy = Math.floor(tile_index / bitmap_tile_width) * tile_size[1];
                    context.drawImage(tile_image, sx, sy, tile_size[0], tile_size[1], x * tile_size[0], y * tile_size[1], tile_size[0], tile_size[1]);
                }
            }
        }

        // Example usage
        const test1_info = JSON.stringify({
            "tile_url": "https://github.com/BeipDev/BeipMU/raw/master/images/Ultima5.png",
            "title": "Overworld Map",
            "tile_size": "16,16",
            "map_size": "10,4",
            "encoding": "hex_4"
        });
        const test1_data = JSON.stringify("0123456789ABCDEF0123456789ABCDEF01234567");

        const test2_info = JSON.stringify({
            "tile_url": "https://github.com/BeipDev/BeipMU/raw/master/images/Ultima5.png",
            "title": "Overworld Map",
            "tile_size": "16,16",
            "map_size": "10,4",
            "encoding": "hex_8"
        });
        const test2_data = JSON.stringify("000102030405060708090A0B0C0D0E0F101112131415161718191A1B1C1D1E1F2021222324252627");

        const test3_info = JSON.stringify({
            "tile_url":"https://github.com/BeipDev/BeipMU/raw/master/images/Ultima5.png",
            "title":"Castle",
            "tile_size":"16,16",
            "map_size":"32,32",
            "encoding":"base64_8"
        });
        const test3_data = JSON.stringify(
"T09PT08FBQUFBQUFBQUFBQUFBQUFBQUFBQVPT09PTwVPpqamTwUFBQUFBQUFBQUFBQUFBQUFBQUFBU+mRERPBU+myKZPT09PT09PT09PT09PT09PT09PT09PT0TIRE8FT0REpk/HRERERERERERERERERERERERERMVPr0SvTwVPT09PT09PT09PTwUFMURERDMFBU9PT09PT09PT09PBQUFT8RPq6ydq6xPBQUxREREMwUFT6usrausT8RPBQUFBQVPRE9ERERERE8FBTFEREQzBQVPRERERK9PRE8FBQUFBU9ET1xdRFxdTwUFMURERDMFBU+rrERERLhETwUFBQUFT0S4RERERERPBQUxREREMwUFT0RERESvT0RPBQUFBQVPRE9ERJEpk08FBTFEREQzBQVPq6ypq6xPRE8FBQUFBU9ET09PT09PTwUFMU9ETzMFBU9PT09PT09ETwUFBQUFT0QFBQUFBQUFBQUFT8ZPBQUFBQUFBQUFBURPBQUFBQVPRAUFBQUFBQUFT09PT09PTwUFBQUFBQUFRE8FBQUFBU9EMjIyMjIyMgVPXF2lXF1PBTIyMjIyMgVETwUFBQUFT0RERERERERPT09ERJBERE9PT0RERERERERPBQUFBQVPRERERERERETFT0REyERET8dERERERERERE8FBQUFBU9ERERERERET09PRERERERPT09ERERERERETwUFBQUFT0QwMDAwMDAwBU+rrEREqU8FMDAwMDAFBURPBQUFBQVPRAVPT09PT08FT09PT09PTwVPT09PT08FRE8FBQUFBU9EBU/IRERETwUFBU/ETwUFBU/IRES/TwVETwUFBQUFT0QFT0SUlZZPT08x+ET4M09PT0SUm5ZPBURPBQUFBQVPRAVPRERERFxdTzFEREQzT0SSRESQRE8FRE8FBQUFBU9EBU9PRERERERPMURERDNPlJyWRERPTwVETwUFBQUFT0RERERERERcXU8xREREM09EkERERERERERPBQUFBQVPRE9E2ERPRERETzFEREQzT1tEW09E2ERPRE8FBQUFBU/GT0RERE9PT09PMURERDNPT09PT0RERE/GTwUFBU9PT09PT09EBQUFBQUxREREMwUFBQUFRE9PT09PT08FT0Svr0/HRERERERERERERERERERERERERMVPr0RETwVPr8hET09PT09PT09PT0RERE9PT09PT09PT0+vyERPBU+vr0RPBQUFBQUFBQUFREREBQUFBQUFBQUFT6+vRE8FT09PT08FBQUFBQUFogVEREQFogUFBQUFBQVPT09PTwUFBQUFBQUFBQUFBQUFBURERAUFBQUFBQUFBQUFBQUFBQ"
        );

        // Uncomment below to simulate receiving the data
        // OnGMCP("tilemap.info", test3_info);
        // OnGMCP("tilemap.data", test3_data);

    </script>
</body>
</html>
````
