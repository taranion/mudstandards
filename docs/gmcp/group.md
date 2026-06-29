---
sidebar_label: group
---
# The ``Group`` package

Source: [Aardwolf](https://www.aardwolf.com/wiki/index.php/Clients/GMCP#aardmodules_group)

## group


Group GMCP messages are only sent once per second and are only  sent then if the group itself has changed or the vitals/stats of a group member have changed. For larger groups this may be once every 2 seconds (10+ players) or every 3 seconds (20+ players). 

The GMCP group information consists of two main sections, the group header and an array of members. The group header contains:

````json
group { 
    "groupname": "lsdkfs", 
    "leader": "Lasher", 
    "created": "28 Dec 14:05", 
    "status": "Private", 
    "count": 2, 
    "kills": 0, 
    "exp": 0, 
    "members": [
        { 
            "name": "Lasher", 
            "info": { 
                "hp": 50054, 
                "mhp": 50054,
                "mn": 65655, 
                "mmn": 65655, 
                "mv": 41629, 
                "mmv": 41629, 
                "align": 2500, 
                "tnl": 43500, 
                "qt": 0, 
                "qs": 0, 
                "lvl": 210, 
                "here": 1 } } , 
        { 
            "name": "Razor", 
            "info": { 
                "hp": 31191, "mhp": 31191,"mn": 6199, "mmn": 6199, "mv": 5775, "mmv": 5775, 
                "align": -2496, "tnl": 790, "qt": 0, "qs": 0, "lvl": 201, "here": 0 } } 
    ] 
}
````
Most of the group information is fairly self explanatory but 'qs' is  worth explaining further - this field indicates what 'qt' (quest time)  actually represents:

- **qt**
  (*Optional*) Quest time<br/>
    0 - Player has no quest timer and can quest. Qt should be zero.<br/>
    1 - Player is questing. Qt represents time left on quest.<br/>
    2 - Player is waiting to quest. Qt represents time until they can quest.<br/>
    3 - Character is a mob, unable to quest.

The "here" field is for whether or not the member is in the same room as the one receiving the GMCP information.

**Note:**  Group information is not sent by default. It needs to be turned on either by:

- Sending 'group on' or 'group off' via  GMCP. If you are using the Aardwolf MUSHclient, this can be achieved  manually by using 'sendgmcp group on'.
