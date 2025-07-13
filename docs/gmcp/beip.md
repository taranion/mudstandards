---
sidebar_label: beip
---
# The ``Beip`` package

Source: [BeipMu](https://github.com/BeipDev/BeipMU/blob/master/Documentation/GMCP.md)

## beip.stats

This command lets you display and update stats in multiple stat windows.
````json
beip.stats {
   "Player":
   {
      "values":
      {
         "0_Name": { "prefix-length": 2, "string": "Bennet", "name-color":"Ansi256(56)" },
         "1_Hit Points": { "prefix-length": 2, "range": { "value":823, "max": 1000, "bar-fill": "#00FF00" }, "value-color": "#345678" },
         "2_Energy Points": { "prefix-length": 2, "range": { "value": 60, "max": 100, "bar-fill": "#8080FF" }, "color":"#FF0000" },
         "3_PP": { "prefix-length":2, "string":"30/60 \u0041 \u2648\u2640 \u849c\u8089" },
         "3_": { "prefix-length":2, "string":"" },
         "4_Money": { "prefix-length": 2, "int":123456, "color":"#FFFF00", "name-alignment":"right" },
         "5_Progress": { "prefix-length": 2, "progress": { "label": "75%", "value":0.75, "fill-color": "#FF0000" } },
         "6_Experience": { "prefix-length": 2, "progress": { "label": "1,234 XP", "value":0.65, "fill-color": "#C07070", "empty-color": "#804040", "outline-color":"transparent" } }
      },
      "background-color": "#002040"
   }
   "Attributes":
   {
      "values":
      {
         "Strength": { "int": 10 },
         "Dexterity": { "int": 5 },
         "Charisma": { "int": 1 },
         "Stamina": { "int": 15 }
      }
   }
}
````

## beip.id

## beip.tilemap
This message will cause the map window to appear and describes all of the properties of it. It is needed before the data message is sent. If a map already exists and a new info message is received, it will change any properties that change (new tileset, etc..) if the map size is modified then the map data is initialized to tile '0' of the new size.
````json
beip.tilemap.info {
  "Map1":
  {
     "tile-url":"https://github.com/BeipDev/BeipMU/raw/master/images/Ultima5.png",
     "tile-size":"16,16",
     "map-size":"10,4",
     "encoding":"Hex_4"
  }
}
````

## beip.tilemap.data
This message holds the map data itself, and must match what the info message has described (more/less data will result in an error). Note that any number of data messages can be sent without a new info message. This makes it easier to update map content when nothing else changes.````json
````json
beip.tilemap.data { "Map1":"0123456701234567012345670123456701234567" }
````
