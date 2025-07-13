---
sidebar_label: MMP
description: MMP (working name) is a specification for MUDs to make room data available to client and/or users for the creation of maps. 
slug: /MMP_Specification
---
# MUD Mapping Protocol

Iron Realms Entertainment (IRE) games are in the process of creating a way to export our in game map data so that clients (or players) can easily access and download this data to use as needed. Rather then creating an IRE specific protocol, it would be much better if this was a universal system for all muds and clients to use and implement.

This page is barely fleshed out, so hopefully over the next few days we can get this looking much more formalized. 

##  Purpose

Significant testing in the IRE introductory tours has shown a  significant drop off of players when they reach portions of the game  where movement is involved. This protocol will allow clients to provide  quick graphical maps for their users. Of course, this means that  introductory clients used for the game (java and flash) should also  provide some kind of basic mapping display as this is where new players  will first experience your game.



##  Map Data

IRE is currently exporting the data in the form of a single XML file  OR as several XML files. Here are some links to the current files from  Imperian.



###  What Data?

What data should we be sending out? Here are my initial thoughts:

- Areas
  - ID Number
  - Name
- Rooms
  - ID Number
  - Area ID
  - Room Title or Brief Desc
  - Room Coordinates
  - Room Exits and attached room
  - Room Features (shop, forge, healer, teacher, etc)
- Environments
  - ID Number
  - Name
  - Color

Areas will allow clients to 



###  Format

Should we continue to use XML to do this, we need to make sure the  format is standard. Here is an example of what we are currently doing.

> ```
> <?xml version="1.0"?>
> <map>
> <areas>
>    <area id="1" name="the ruins of Caanae" />
>    <area id="2" name="the Khuno Acropolis" />
> 
>    ... areas
> 
>    <area id="194" name="Alcine Estate" />
>    <area id="195" name="Abandoned Tower" />
> </areas>
> <rooms>
>    <room id="8463" area="1" title="Aryana's Spring" environment="8">
>       <coord x="0" y="0" z="0" />
>       <exit direction="north" target="8472" />
>       <exit direction="east" target="8458" />
>       <exit direction="south" target="8453" />
>       <exit direction="west" target="8465" />
>       <exit direction="down" target="8558" />
>       <exit direction="in" target="17200" />
>    </room>
> 
>    ... more rooms
> 
>    <room id="8472" area="1" title="The empty northern square" environment="8">
>       <coord x="0" y="1" z="0" />
>       <exit direction="northeast" target="8519" />
>       <exit direction="southeast" target="8458" />
>       <exit direction="south" target="8463" />
>       <exit direction="southwest" target="8465" />
>       <exit direction="in" target="7172" />
>    </room>
> </rooms>
> <environments>
>    <environment id="1" name="Dark Forest" color="2" />
>    <environment id="2" name="Constructed underground" color="3" />
> 
>    ... more environments
> 
>    <environment id="78" name="Scrublands" color="2" />
>    <environment id="79" name="Tower" color="7" />
> </environments>
> </map>
> ```

###  Examples

- **All Area, Room, and Environment Data** [http://www.imperian.com/maps/map.xml](https://web.archive.org/web/20100330083116/http://www.imperian.com/maps/map.xml)
- **The Northern Celidon** [http://www.imperian.com/maps/10.xml](https://web.archive.org/web/20100330083116/http://www.imperian.com/maps/map.xml) 
- **The Waelin River** [http://www.imperian.com/maps/12.xml](https://web.archive.org/web/20100330083116/http://www.imperian.com/maps/map.xml) 
- **Towne of Tirhin** [http://www.imperian.com/maps/106.xml](https://web.archive.org/web/20100330083116/http://www.imperian.com/maps/map.xml)

##  Problems

The following may or may not be problems, based on how you feel maps should be displayed.

###  Exploration

Part of the fun of playing a new game is exploring the world,  learning about new places, and discovering their treasures. One thing we should try to avoid is simply dumping every single room instantly out  to the new player. Players who have been around for a while will not  mind this, as they know the world already. 

This is probably something that would mainly need to be addressed client side, in how those rooms are hidden, revealed, and displayed.

###  Overlapping Rooms

Some people will consider this a problem, others will not. IRE does  not like overlapping rooms. It seems most clients do not care. I think  it will be up to games to export their data as they want those maps  displayed.

------

**Source:** [Internet Archive](https://web.archive.org/web/20100330083116/http://www.mudstandards.org/MMP_Specification) of original mudstandards.org
Likely from *Tomas Mecir* of Iron Realms
