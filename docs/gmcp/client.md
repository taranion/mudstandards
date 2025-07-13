---
sidebar_label: client
---
# The ``Client`` package

This package has been defined by Mudlet.

Source: [Mudlet Wiki](https://wiki.mudlet.org/w/Manual:GMCP_Extensions)

## Client.GUI
Sent by the server to inform the client where an extension package for the MUD server can be found.
````json
Client.GUI {
  "version": "39",
  "url": "http://www.stickmud.com/mudwww/StickMUD.mpackage"
}
````
- **url**
  (*Mandatory*) Download URL
- **Version**
  (*Mandatory*) Version number of the download. May be a string or an integer value.


## Client.Map

Mudlet can download a map for the user as soon they open the mapper for the first time, as well as re-download the map from the Mapper tab in settings.
To let Mudlet know where it can download your map, send the following after GMCP has been enabled:

````json
Client.Map {
  "url": "https://..."
}
````
- **url**
  (*Mandatory*) URL to an MMP standard map (indicated by XML ending) or other file formats the client understands.