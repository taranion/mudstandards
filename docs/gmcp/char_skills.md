---
sidebar_label: char.skills
---
# The ``Char.Skills`` package

:::note
Source https://nexus.ironrealms.com/GMCP#Char.Skills
:::

## Char.Skills.Get 

Sent by the client to the server.
Request for the server to send the list of items located inside another item.
````json
Char.Skills.Get {
    "group": "<skillgroup>",
    "name": "<skillname>"
}
````
- If both group and name is provided, the server will send Char.Skills.Info for the specified skill
- If group is provided but name is not, the server will send Char.Skills.List for that group
- Otherwise the server will send Char.Skills.Groups

## Char.Skills.Group
Sent by server on request or at any time (usually if the list changes)
````json
Char.Skills.Group [
  {
    "name": "Perception",
    "rank": "Transcendent (100%)"
  },
  {
    "name": "Survival",
    "rank": "Inept (0%)"
  },
  {
    "name": "Weaponry",
    "rank": "Inept (0%)"
  },
  {
    "name": "Tattoos",
    "rank": "Inept (0%)"
  },
  {
    "name": "Evasion",
    "rank": "Inept (0%)"
  },
  {
    "name": "Engineering",
    "rank": "Inept (0%)"
  },
  {
    "name": "Taming",
    "rank": "Inept (0%)"
  },
  {
    "name": "Concoctions",
    "rank": "Inept (0%)"
  },
  {
    "name": "Toxins",
    "rank": "Inept (0%)"
  },
  {
    "name": "Smithing",
    "rank": "Inept (0%)"
  },
  {
    "name": "Malignosis",
    "rank": "Adept (1%)"
  },
  {
    "name": "Necromancy",
    "rank": "Inept (0%)"
  },
  {
    "name": "Evileye",
    "rank": "Adept (40%)"
  }
]
````
- For IRE games, groups are skills like Survival or Elemancy
- Message body is an array of objects, each being one name and the skill rank

## Char.Skills.List 

Sent by the server on request only. List of skills in a group available to the character

````json
Char.Skills.List {
  "group": "Elemancy",
  "desc": [
    "Cast light",
    "Make your skin hard as stone",
    "Cast a bold of fire"
  ],
  "list": [
    "Light",
    "Stoneskin",
    "Firelash"
  ]
}
````
- For IRE games, this is the list visible on AB &lt;groupname&gt;
- Message body is an object with keys group, desc and list, where group is the group name as a string
- The desc value is the description of the skill as seen in AB &lt;groupname&gt;
- The list value is an array of strings, each being the name of one skill
- Note for IRE games This will return all skills in the group, even not learned ones.



## Char.Skills.Info 
Information about a single skill, only sent upon request
````json
Char.Skills.Info {
  "group": "Elemancy",
  "skill": "Firelash",
  "info": "blah blah"
}
````
- Message body is an object, keys are group, skill, and info, values are strings
- Group and skill identify the request, info is a description (usually multi-line) of the skill's functionality and usage
- Note for IRE games Unlearned abilities will have *** You have not yet learned this ability *** in their info field in addition to the description
