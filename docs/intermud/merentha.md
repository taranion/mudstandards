---
sidebar_label: Merentha InterMUD System)
---
# Merentha InterMUD System (MIS) v1.2.0-A

## Network Architecture
The Merentha "Intermer" network is a full mesh of hosts that communicate via UDP.

## Features
* Pinging a host
* Who - Get a (formatted) list of users from the remote host
* Finger - Get information about a user
* Tell - send a message to a user
* Creator channel - send messages to admins of a remote server


## Message Encoding
Messages are plain ASCII strings that consist of key/value pairs. Sometimes those values are key value pairs in itself. Than it gets nasty.
* In the top level, pairs are divided by the string ``||M||``. Inside a key/value pair, key and value are divided by ``:||:||:``
* On the medium level, pairs are divided by the string ``||MM||``. Inside a key/value pair, key and value are divided by ``::||::||::``
* On the lowest level, pairs are divided by the string ``||MMM||``. Inside a key/value pair, key and value are divided by ``:::||:::||:::``

*Example 1: Ping Request*
```
name:||:||:GraphicMUD||M||port:||:||:4000||M||udp_port:||:||:10006||M||mudlib:||:||:GraphicMUD||M||driver:||:||:Java 24.0.1+9||M||service:||:||:ping_request||M||known_muds:||:||:1
```
*Example Ping Reply*: The mudlist is a level 3 encoding (see the ``||MMM||``)
```
udp_port:||:||:10006||M||service:||:||:ping_reply||M||driver:||:||:FluffOS v2017.2019||M||name:||:||:Merentha||M||port:||:||:10000||M||mudlib:||:||:Merentha/NM||M||args:||:||:Merentha-Builders::||::||::udp_port:::||:::||:::10007||MMM||pings:::||:::||:::1||MMM||address:::||:::||:::50.22.147.227||MMM||driver:::||:::||:::FluffOS v2017.2019||MMM||name:::||:::||:::Merentha-Builders||MMM||port:::||:::||:::10001||MMM||mudlib:::||:::||:::Merentha/NM||MM||GraphicMUD::||::||::udp_port:::||:::||:::10006||MMM||pings:::||:::||:::0||MMM||address:::||:::||:::212.60.194.176||MMM||driver:::||:::||:::Java 24.0.1+9||MMM||name:::||:::||:::GraphicMUD||MMM||port:::||:::||:::4000||MMM||mudlib:::||:::||:::GraphicMUD||M||remote_user:||:||:0
```

## Messages for services

### ping_request
This request announces your MUDs presence to the remote MUD. The response contains not only information about used Mudlib and driver, but also a list of other known MUDs.

The original implementation sends ping requests in a 15 minute interval and repeats them twice if needed.

| Parameter  | type    | Example            | Description                                            |
| ---------- | ------- | ------------------ | ------------------------------------------------------ |
| name       | String  | Merentha           | The name of your MUD. Should not contain spaces        |
| port       | Integer | 10000              | Port players use to connect                            |
| udp_port   | Integer | 10006              | Port of the MIS service (you should already know this) |
| mudlib     | String  | Merentha           | Name of the Mudlib                                     |
| driver     | String  | FluffOS v2017.2019 | Type and version of the Mudlib                         |
| service    | String  |                    | Always "ping_request"                                  |
| known_muds | String  | Integer            | Number of MIS-connected MUDs your server is aware of   |

```
name:||:||:GraphicMUD||M||port:||:||:4000||M||udp_port:||:||:10006||M||mudlib:||:||:GraphicMUD||M||driver:||:||:Java 24.0.1+9||M||service:||:||:ping_request||M||known_muds:||:||:1
```


### ping_response

### rwho_request

### rwho_response

### rfinger_request

### rfinger_response

### rtell_request
Tell 

### rtell_response

### rcre_request
Send a message on the Coder/Creater channel


### rcre_response


MIS is designed to connect all Merentha MudLib MUDs together.  It uses UDP
datagram sockets, extreamly similar to I2 (InterMUD2).  In fact with minor
modifications the new network daemon can be converted to use with I2
instead of MIS.

There are several reasons why I chose to create MIS instead of just
connecting the Merentha Lib to I2 (or I3).
* I have no experience with I3
* There is lots of spam on I2 already
* This is a learning project for myself above all else
* MIS is designed to promote and further develop the Mer Lib

All things considered I learn't more by attempting to create my own
InterMUD System then just hooking up to an existing one.

By default the UDP port is your MUDport +6 (I2 uses +8).  Since encoding
of information on I2 is different then on MIS I have used a seperate port,
this also enable people to incorperate both InterMUDs if they wish.  I do
not know what I3 uses.  It should be noted that in reality both I2 and MIS
could interact on port +8 together and have your daemons decode which is
which, but I didn't want that.



```c
mapping convert_string_to_map(string str) {
    mapping tmp=([]);
    string *parts=explode(str,"||M||");
    string key, val;
    int i=sizeof(parts);
    while(i--) {
    sscanf(parts[i], "%s:||:||:%s", key, val);
    tmp[key]=val;
    }
    return tmp;
}
mapping convert_string_to_map2(string str) {
    mapping tmp=([]);
    string *parts=explode(str,"||MM||");
    string key, val;
    int i=sizeof(parts);
    while(i--) {
    sscanf(parts[i], "%s::||::||::%s", key, val);
    tmp[key]=val;
    }
    return tmp;
}
mapping convert_string_to_map3(string str) {
    mapping tmp=([]);
    string *parts=explode(str,"||MMM||");
    string key, val;
    int i=sizeof(parts);
    while(i--) {
    sscanf(parts[i], "%s:::||:::||:::%s", key, val);
    tmp[key]=val;
    }
    return tmp;
}
```

```c
    message=convert_string_to_map(message);
    sscanf(socket_address(s), "%s %s", junk, args);
    if(junk=="0.0.0.0") sscanf(host, "%s %s", junk, args);
    if(mapp(message)) {
    if(!message["name"]) { return; }
    if(!message["service"] || member_array(message["service"],__Services)==-1) { return; }
    __MUDS[message["name"]]=([
      "name":message["name"],
      "address":junk,
      "udp_port":message["udp_port"],
      "port":message["port"],
      "mudlib":message["mudlib"],
      "driver":message["driver"],
      "pings":0,
    ]);
    args=call_other(this_object(),({message["service"],message}));
    if(!args) { return; }
    if(strsrch(message["service"],"_request")!=-1) {
        open_socket_to_send(""+junk+" "+message["udp_port"],
          (["name":mud_name(),
        "port":query_host_port(),
        "udp_port":MERENTHA_INTERMUD,
        "mudlib":mudlib(),
        "driver":driver(),
        "remote_user":message["remote_user"],
        "args":args,
        "service":replace_string(message["service"],"_request","_reply"),
          ]) );
        return;
    }
```