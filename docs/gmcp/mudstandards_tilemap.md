---
sidebar_label: mudstandards.tilemap
---

# The 'mudstandards.tilemap' package

The purpose of this package is to allow MU\* servers to control coordinate based maps. It is inspired from the `beip.tilemap` ([link](https://github.com/BeipDev/BeipMU/blob/master/TileMap.md)) package, but adds features to work with multiple tilesets per map, layering and options to choose tileset resolutions.

## Initial detection (TODO)

Query
Support   (graphic|ansi)

## mudstandards.tilemap.tilesets

With this command the server informs the client about all tilesets available and assigns identifiers to reference them in later commands. Tilesets may come in different resolutions - the client is free to pick the one suited best.

```json
mudstandards.tilemap.tilesets {
    "terrain": {
    	url: "http://.../SmallTiles.png",
        sizeX: 32,
        sizeY: 32,
        anim: {
        	"50": 4,
        	"54": 4
        }
    },
	"monster": {
        url: "https://...",
        sizeX: 32,
        sizeY: 32,
        anim: {
            "0": "4",
            "4": "4"
        }
    }
  }
}
```

Each tileset is named and the name is later used to reference it from other commands. Also tilesets may come with different tile sizes: 8, 16 or 32 - refererring to the width and height of a single tile.

- **size**  - Width x Height

- **url** - URL where to download the tileset

- **anim-frames** - Those tiles that are animated, are referenced by their number and have the number of animation frames assigned.

  > [!IMPORTANT]
  >
  > Tiles not listed in anim-frames are considered to consist of only 1 frame, meaning that they are not animated.
  >
  > A client not supporting animation can just ignore animation frames.



## tilemap.area

This command tells the client to prepare one or more areas that should show tilemaps. Each area can make use of all tilesets that have been uploaded by `tilemap.tilesets` before. If the tilesets are provided in multiple resolutions, it is up to the client to pick the one used - e.g. because the user chooses a resolution or because of the screen size.

```json
mudstandards.tilemap.area {
    "World Map": {
        "map-size": "21x21",
        "encoding": "Hex_8",
        "tilesets": {
            "terrain": "0",
            "monster": "128"
        },
        frame: "world"
    },
    "Surrounding": {
        "map-size": "11x11",
        "encoding": "Hex_16",
        "tilesets": {
            "terrain": "0",
            "objects": "128",
            "monster": "256"
        }
    }
}
```

For each defined area, there is a **map-size**, telling the client how many tiles wide and high the area will be - and an **encoding** to tell the client how many hexadecimal digits each tile number will have. Valid values are **Hex_8**, **Hex_16** and **Hex_24** for 8, 16 or 24 bit / resp. 1,2 or 3 digits.

**encoding** - Encoding format of the content data (the 012345... part between the tags) encoding has multiple possibilities to make it easy to use or as compact as possible:

- Hex_4 - Hexadecimal with 4 bits per tile (16 tiles possible). In this format a single hex character is a single tile. ** The tilemap tag at the start of this document uses hex_4 encoding
- Hex_8 - Two hex digits (256 tiles possible)
- Hex_16 - Four hex digits (2 bytes) per tile (65536 tiles possible)

**frame** -(string) (optional) - If by some other means windows or frames have been defined,  this optional parameter can be used to refer to a frame where this map shall be displayed in. In such a case, it usually replaces all existing content in that frame.

To prepare the interpretation of the tile number, the **tilesets** element tells the client, from which tileset a tile must be taken. E.g. in the "Surrounding" example above, a tile number of 160 would be the 32. tile in the "objects" tileset.

## tilemap.data

This command will be send for each update in the map. The identifier tells *which* area is updated. For each area there are one or more layers given. Drawing should start with the layer with the lowest number and then processed ascending. For each layer, there are all tile numbers for the whole map expected. E.g. if a map is 11x11 tiles large and have a HEX_24 encoding, each layer must contain 11x11x3 = 363 hexadecimal digits.

```json
mudstandards.tilemap.data {
	"Surrounding": {
		"0": "<string of layer data, depending on encoding",
		"1": "<string of layer data, depending on encoding"
	}
}
```

------

[Beip.Tilemap]: https://github.com/BeipDev/BeipMU/blob/master/TileMap.md	"Beip.Tilemap Package"

