---
sidebar_label: Portal MIP
---
# Portal© for Windows MIP v9.0
(Messaging Information Protocol)

You can find the original specification [here](https://www.gameaxle.com/PortalMIP9.pdf)

## Overview
Portals MIP is an in-band protocol. To distinguish regular content from 
protocol content, the server needs the character sequence ``#K%`` at the start 
of each protocol line.

### Detection
After a connection has been established, the Portal client scans server 
output for the word ``Welcome``, like in *Welcome to SuperMUD*.
If the trigger word has been detected, the Portal client sends a line like this:
```
3klient 10401~Portal3.500
```
| Output  | Type          | Description                                                  |
| ------- | ------------- | ------------------------------------------------------------ |
| 3klient | Fixed keyword | Command name for the jumpstart phase <br />("3klient" was an previous name or the Portal client) |
| 10401   | &lt;SEC_CODE&gt; Random 5 digit session PIN                                   |
| 3.500   | Version       | Versionnumber of the client                                  |

## Commands

Commands send to the client look like this

```
#K%00005008AAC12:46
```

Explanation:

| Value | Explanation                        |
| ----- | ---------------------------------- |
| #K%   | The ACTIVATE literal               |
| 00005 | The security code / session PIN    |
| 0008  | The number of characters to follow |
| AAC   | The command                        |
| 12:46 | Data for the command               |

> **ATTENTION**<br/>
> The command list in this document is incomplete. Consult the [original documentation](https://www.gameaxle.com/PortalMIP9.pdf) for more commands


### Send Sound (AAA)

This literal is used to send the filename of a sound that you wish Portal© to play. All sounds are
.WAV files and are stored in the media/sounds/ directory beneath the dir where the Portal©
executable resides.
Range for this literal is restricted by Windows’ file naming conventions. As a general rule, you
should not use anything but alphanumeric characters (a, b, c, 1, 2, 3) and the underscore (_).
The filename is not case-sensitive. The length of the total filename cannot be longer than 255
characters.

> This function is only enabled if the user has not disabled system sounds.


**Format**: ``ACTIVATE+SEC_CODE+CHAR_COUNT+CL_SEND_SOUND+filename``

Example: filename = lock
```
#K%00005007AAAlock
```
### Send Image (AAB)

This is a dual data point literal.
This literal is virtually identical to the CL_SEND_SOUND literal except that it is used to send the
filename of the image to be displayed in the Portal© Imagery. All images must be either bitmaps
or gifs (.BMP and .GIF extensions respectively). You do not need to include the file extension
unless you wish to specify. Portal© will look for a bitmap and then a gif, in that order. All images
are stored in the media/images/ directory. The imagelabel (which will appear below the image in
the Imagery) is optional, but the CL_DELIM is not. If you don't want a label, simply end the string
with CL_DELIM.

**Format**: ``ACTIVATE+SEC_CODE+CHAR_COUNT+CL_SEND_IMAGE+filename+CL_DELIM+imagelabel``

Example: `filename = grey_elf`, `imagelabel = Gray Elf`
```
#K%00005020AABgrey_elf~Gray Elf
```

### Send Reboot (AAC)

This is a multiple data point literal.
This literal is similar to the CL_SEND_SOUND except that it is used to send the filename of the
music file to be played. Valid MIDI files formats are supported (*.MID, .RMI, etc.) as well as MP3
files (.MP3) You do not need to include the file extension unless you wish to specify. All music
files are stored in the media/sounds/ directory. You can also choose the number of iterations to
play of the file. The file will play iterations # of times. If you don’t specify a number of
iterations, the file will play only once.

> Note: This function is only enabled if the user has not disabled remote media support.

**Format**: ``ACTIVATE+SEC_CODE+CHAR_COUNT+CL_SEND_MIDI+filename+CL_DELIM+iterations``

(non-repeating music example)
filename = canon_in_D ,iterations = &lt;nothing&gt;
```
#K%00005013AADcanon_in_D
```

(repeating music example – plays it three times)
filename = canon_in_D~1 ,
iterations = 3
```
#K%00005015AADcanon_in_D~3
```

(stops the currently playing music file)
filename =  &lt;nothing&gt;
iterations =  &lt;nothing&gt;
```
#K%00005003AAD
```

### Download Media (AAH)

his is a multiple data point literal.
This literal allows you to automatically download media files to the user's media\sounds and
media\images directories. The available files for download must be of the kind .WAV, .MID, .RMI,
.MP3 (sounds) .BMP, .GIF or .AVI (images). It requires that you specify a filename, which will be
the filename that will appear in the user's applicable media directory. You also must supply the
URL where the media file resides on the web (usually on your MUD's site somewhere).

**Notes:**

- If the filename already exists in the user's applicable media directory, this operation aborts.
  Basically it's a good idea to keep your files named uniquely, maybe prepended with your
  MUD's name (e.g. blahmud_bigroar.wav).
- The filename cannot contain spaces and is limited to 255 characters in length.
- The filename must contain the file type extension (.GIF, .AVI etc.)
- The filename is not case sensitive, but the URL is.
- The URL is case sensitive, in case you missed it.
- This function is only enabled if the user has not disabled auto downloading.
- The user will be notified of the filename that was downloaded to their applicable media
  directory, as well as the URL from which it was downloaded.
- Try to keep the files relatively small (under 100K) as downloading them sucks extra
  bandwidth from the user. Also, spread them out if you can. Having the user download 100
  files one right after the other is not always a good idea. A good possible usage would be a
  command to use on your MUD which would download a list of specific media for the user.
  This way they could enter the command and come back in a few minutes when it’s
  downloaded. Portal© won’t attempt to download files they already have, so it’s ok to send
  files they don’t have.
- Unless you KNOW your users have a killer connection, don’t even think about sending them
  .MP3 files.

**Format**: ``ACTIVATE+SEC_CODE+CHAR_COUNT+CL_DOWNLOAD_SOUND+filename+
CL_DELIM+ URL``

(MIDI example)
filename = B5.rmi
URL = http://www.gameaxle.com/B5.rmi
```
#K%00005040AAHB5.rmi~http://www.gameaxle.com/B5.rmi
```

(WAV example)
filename = lock.wav
URL = http://www.gameaxle.com/lock.wav
```
#K%00005044AAHlock.wav~http://www.gameaxle.com/lock.wav
```
