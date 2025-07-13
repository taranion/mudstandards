---
sidebar_label: client.media
---
# The ``Client.Media`` package

This package can be used to play sound effects, background music or videos on the client.
It copies concepts from the MUD Sound Protocol (MSP) and extends it further.

Source: [Mudlet Wiki](https://wiki.mudlet.org/w/Standards:MUD_Client_Media_Protocol)

## Client.Media.Default
Sent by the server to identify to the game client a default URL directory to load media files from an external resource.<br/>
**Guidance**: For games that automatically download media files, perform a Client.Media.Default GMCP event once upon player login.

````json
Client.Media.Default {"url": "https://www.example.com/media/"}
````
| Key  | Required | Value       | Purpose                                                      |
| ---- | -------- | ----------- | :----------------------------------------------------------- |
| url  | Maybe    | *url*       | Resource location where the media file may be downloaded. <br />Last character must be a / (slash). <br />Only required if a url was not set with Client.Media.Default. |

## Client.Media.Load

Load media files from an external source.<br/>
**Guidance**: For games that automatically download media files and have the capability to cache with the game client.
````json
Client.Media.Load {
  "name": "sword1.mp3",
  "url": "hxxps://www.example.com/media/"
}
````
| Key  | Required | Value       | Purpose                                                      |
| ---- | -------- | ----------- | :----------------------------------------------------------- |
| name | Yes      | *file name* | Name of the media file. <br />May contain directory information (i.e. weather/lightning.mp3). |
| url  | Maybe    | *url*       | Resource location where the media file may be downloaded. <br />Last character must be a / (slash). <br />Only required if a url was not set with Client.Media.Default. |

## Client.Media.Play
Play media files.<br/>
**Guidance**: Game clients could choose whether to play only one media file at one time or multiple files at one time.
````json
Client.Media.Play {
  "name": "80_Blacksmith_Shoppe.mp3",
  "url": "https://www.example.com/media/",
  "type": "music",
  "tag": "environment",
  "volume": 25,
  "fadein": 5000,
  "fadeout": 7000,
  "start": 1000,
  "finish": 20000,
  "loops": 3,
  "priority": 60,
  "continue": true,
  "key": "area-background-music",
  "caption": "Blacksmith Hammering"
}
````

| Key        | Required | Value                       | Default | Purpose                                                      |
| ---------- | -------- | --------------------------- | ------- | :----------------------------------------------------------- |
| "name"     | Yes      | *file name*                 |         | Name of the media file.<br/> May contain directory information (i.e. weather/lightning.mp3). |
| "url"      | Maybe    | *url*                       |         | Resource location where the media file may be downloaded.<br/>Last character must be a / (slash).<br/>Only required if the file is to be downloaded remotely and a url was not set above with Client.Media.Default or Client.Media.Load. |
| "type"     | No       | "sound", "music" or "video" | "sound" | Identifies the type of media.                                |
| "tag"      | No       | *tag*                       |         | Helps categorize media.                                      |
| "volume"   | No       | 1 to 100                    | 50      | Relative to the volume set on the player's client.           |
| "fadein"   | No       | *msec*                      |         | Volume increases, or fades in, ranged across a linear pattern from one to the volume set with the "volume" key.<br/>Start position: Start of media.<br/>End position: Start of media plus the number of milliseconds (msec) specified.<br/>1000 milliseconds = 1 second. |
| "fadeout"  | No       | *msec*                      |         | Volume decreases, or fades out, ranged across a linear pattern from the volume set with the "volume" key to one.<br/>Start position: End of the media minus the number of milliseconds (msec) specified.<br/>End position: End of the media.<br/>1000 milliseconds = 1 second. |
| "start"    | No       | *msec*                      | 0       | Begin play at the specified position in milliseconds.<br/>1000 milliseconds = 1 second. |
| "finish"   | No       | *msec*                      | 0       | End play at the specified position in milliseconds.<br/>1000 milliseconds = 1 second. |
| "loops"    | No       | -1, or >= 1                 | 1       | Number of iterations that the media plays.<br/>A value of -1 allows the sound or music to loop indefinitely. |
| "priority" | No       | 1 to 100                    |         | Halts the play of current or future played media files with a lower priority while this media plays. |
| "continue" | No       | true or false               | true    | Continues playing matching new music files when true.<br/>Restarts matching new music files when false. |
| "key"      | No       | *key*                       |         | Uniquely identifies media files with a "key" that is bound to their "name" or "url".<br/>Halts the play of current media files with the same "key" that have a different "name" or "url" while this media plays. |
| "caption"  | No       | *text*                      |         | [Caption-friendly](https://dcmp.org/learn/602-captioning-key---sound-effects-and-music%7CDCMP) textual representation of the media, such as  onomatopoeia or sound cues (e.g., `thunderclap`, `whoosh`), enabling  accessibility.> |

## Client.Media.Stop

Stop playing media files.
**Guidance**: An empty body will stop all media.

````json
Client.Media.Stop {
  "name": "city.mp3",
  "type": "music",
  "tag": "environment",
  "priority": 60,
  "key": "area-background-music",
  "fadeaway": true,
  "fadeout": 7000
}
````

| Key        | Required | Value                       | Default | Purpose                                                      |
| ---------- | -------- | --------------------------- | ------- | :----------------------------------------------------------- |
| "name"     | No       | *file name*                 |         | Stops playing media by name matching the value specified.    |
| "type"     | No       | "sound", "music" or "video" | "sound" | Stops playing media by type matching the value specified.    |
| "tag"      | No       | *tag*                       |         | Stops playing media by tag matching the value specified.     |
| "fadeaway" | No       | *msec*                      |         | Decrease volume from the current position for a given duration, then stops the track.<br/>Given duration is the lesser of the remaining track duration or the fadeout specified in Client.Media.Play.<br/>If fadeout was not specified in Client.Media.Play, then the optional fadeout parameter from Client.Media.Stop or a default of 5000 milliseconds will be applied. |
| "fadeout"  | No       | *msec*                      |         | Default duration in milliseconds to decrease volume to the end of the track.<br/>Only used if fadeout was not defined in Client.Media.Play. |
| "priority" | No       | 1 to 100                    |         | Stops playing media with priority less than or equal to the value. |
| "key"      | No       | *key*                       |         | Stops playing media by key matching the value specified.     |

