---
sidebar_label: Circle Intermud
description: One of the first attempts on inter-mud communication
---
# Circle Intermud

:::note
**Source**: [http://mud.stack.nl/intermud/circle.protocol.html](https://web.archive.org/web/20051229142240/http://mud.stack.nl/intermud/circle.protocol.html)<br/>
**Source 2**: [Path for Circle MUDs](https://www.circlemud.org/pub/CircleMUD/contrib/imc/intermud_patch.tar.gz)<br/>
**Source 3**: [Comment regarding closed beta, Dec. 1995](https://www.circlemud.org/maillist/1995-12/0061.html)<br/>
**Source 4**: [Re-Release of Intermud 0.53 from 1996](https://www.circlemud.org/maillist/1996-04/0610.html)<br/>
**Source 5**: [Release of Intermud 0.54 from June 1996](https://www.circlemud.org/maillist/1996-06/0368.html)<br/>
**Source 7**: [Release of Intermud 0.55 from June 1996](https://www.circlemud.org/maillist/1996-06/0382.html)<br/>
**Source 8**: [New Bootmaster IP, Feb 1997](https://www.circlemud.org/maillist/1997-02/0703.html)<br/>
An other intermud protocol was developed by **Stryker@Tempus** (Chris Austin), and is mostly used on Circle-MUDs. However Merc- and Envy-MUDs are supported as well. The protocol is designed to use a stand-alone daemon for communication with other daemons over UDP and communication with the muddriver using a unix socket. It is quite simple to add support directly to the driver or mudlib though, so there is no reason why LPMUDs shouldn't support this protocol though.

For the rest it was pretty much the same as Intermud-2, however the message fields were indexed by their order in the message rather then by field names. Every message has its own command number which indicates the type of this message. In general these packets are definately shorter than an Intermud-2 message.

Stryker started with version 0.00, however the first only released versions are 0.52-0.56 and finally version 1.0. This was developed in the years 1995-1996 and nothing seems to have been one on this since. Because of this, the protocol was later nicknamed Intermud 1(?!)
:::

## *Introduction*
------

The protocol was introduced to the CircleMUD mailing list in 1995, when Stryker (Chris Austin) from TempusMUD was [looking for Alpha testers](https://www.circlemud.org/maillist/1995-08/0100.html).
At that time it likely was the first Intermud protocol for Diku/Circle/Merc/Envy-MUDs 
and it received some interest - so much interest that 4 months later when more
admins tried to join the network without being invited to the Alpha test, 
[Stryker tried to slow that down](https://www.circlemud.org/maillist/1995-12/0061.html) and pointed out that it is still in a controlled closed test group. 

Finally in April 1996 a first general availability was [announced](https://www.circlemud.org/maillist/1996-04/0610.html). As far as I can tell,
other protocols started to get ported to Circle/Diku/Merc/Ency. A [post](https://www.circlemud.org/maillist/1996-07/0721.html)) from July 1996 hints that at least IMC(2?) was available for Circle.

Circle Intermud relied on a central "Bootmaster" server to inform all MUDs of other MUDs, so that they could communicate directly. In early 1997 that Bootmaster changed and the change was not communicated on the CircleMUD mailing list. In the following months more admins asked for help for an unresponsive bootmaster. Later that was improved with a locally stored cache of the MUD list as a fallback. 

The overall interest in *Circle Intermud* seems to have been low, since there was no traffic on the dedicated mailing list.
Also Stryker did not appear on the CircleMUD mailinglist again, so it is safe to say that Stryker as well as the MUDs abandoned the software.

Later in 1998 Christian Loth (from *Realm of Magic*) tried to rattle interest in an evolution of the software he named "*[RoM Intermud](rom_intermud)*".

## Protocol design
The protocol is usually implemented in 2 stages. Some functions are defined in the driver, some are defined in an external intermud program. This daemon communicates with the other daemons via udp and with the muddriver via a local unix socket. Some requests from the mud can be handled directly by the daemon, some are relayed to other daemons (and vice versa).

- Every message sent is printable strings.
- Every message consists of a 'command' and 'parameters'. The set of parameters is fixed for each command.
- Default Port was either 8989 or 9000, but MUDs were free to change that
- The last documented bootmaster (in early 1997) was ``129.59.205.171 2121``

The typical message string looks like:
```
command|param1|param2|param3|
```
or in a slightly more formal description:

```
	message 		::= command '|' parameterlist

	command 		::= Digit Digit Digit Digit

	parameterlist 		::= /* empty */
			  	    | value '|' parameterlist

	value 			::= String
```

Where digit is any number 0 through 9, and 'String' is any printable text not containing the '|' symbol combinations.

New commands can be added without problems for backwards compatiblilty; whoever it is not possible to add extra parameters to a command.

## Command types

The first digit indicates the type of the command:

- 1xxx

  These are general support commands: startup, ping, mudlist.

- 2xxx

  These indicate the normal request/reply messages: tell, page, who.

- 3xxx

  For messages sent to all MUDs at once: wiz.

- 4xxx

  Driver - local daemon communication commands: mudinfo, stats, debug.

## Current protocol

Only a few commands has been implemented so far. The commands I know are [listed here](https://web.archive.org/web/20051217012619/http://mud.stack.nl/intermud/circle.html).

Here is a list of all services used by the Circle Intermud protocol. This list is NOT yet complete, nor is it completely accurate: I'm still working on this bit...



### startup (1000)

Notify the world that we are alive.  	 

* 1000 The Intermud UDP port 
* The name of our mud 
* The mud's login port (TCP) 
* An integer indicating the type of this mud  (1 = Circle, 2 = Merc, 3 = Envy, 4 = Other) 
* The driver version of this mud Intermud version (usually "1.0") 
* Supported services (usually "F0000000") 

### ping request (1020)

Send out information about ourselves. 

1. 1020 The Intermud UDP port 
2. The name of our mud 
3. The mud's login port (TCP) 
4. An integer indicating the type of this mud  (1 = Circle, 2 = Merc, 3 = Envy, 4 = Other) 
5. The driver version of this mud Intermud version (usually "1.0") 
6. Supported services (usually "F0000000") 

### ping reply (1030)

Send out information about ourselves. 	 

1. 1030 
2. The name of our mud The mud's login port (TCP) #
3. The Intermud UDP port 
4. An integer indicating the type of this mud  (1 = Circle, 2 = Merc, 3 = Envy, 4 = Other) 
5. The driver version of this mud Intermud version (usually "1.0") 
6. Supported services (usually "F0000000") 

### mudlist request (1040)

Retrieve a mudlist.

| Position | Parameter | Type    | Description          |
| -------- | --------- | ------- | -------------------- |
| 1        | "1040"    | Integer | Command code         |
| 2        | Player    | String  | Player who is asking |

### mudlist reply (1050)

Send out information about ourselves. 

| Position | Parameter | Type    | Description          |
| -------- | --------- | ------- | -------------------- |
| 1        | "1000"    | Integer | Command code         |
| 2        | Service   | String  | ?          |
| 3        | Info      | String  | ?         |

1. 1050 
2. The name of a mud 
3. The mud's login port (TCP) 
4. The Intermud UDP port 
5. An integer indicating the type of this mud  (1 = Circle, 2 = Merc, 3 = Envy, 4 = Other) 
6. The driver version of this mud Intermud version (usually "1.0") 
7. Supported services (usually "F0000000") ...  (etc.) 

### dns update (1070)

Send out information about ourselves. (Possibly as a reply to a mudlist.) 

1. 1070 
2. The name of our mud 
3. The mud's login port (TCP) 
4. The Intermud UDP port 
5. The driver version of this mud Intermud version (usually "1.0") 
6. An integer indicating the type of this mud  (1 = Circle, 2 = Merc, 3 = Envy, 4 = Other) 
7. Supported services (usually "F0000000") 

### info (1100)

Send out general information for logging purposes only. 

| Position | Parameter | Type    | Description          |
| -------- | --------- | ------- | -------------------- |
| 1        | "1000"    | Integer | Command code         |
| 2        | Service   | String  | ?          |
| 3        | Info      | String  | ?         |

### tell request (2000)

Send out information about ourselves. 	 

| Position | Parameter | Type    | Description          |
| -------- | --------- | ------- | -------------------- |
| 1        | "2000"    | Integer | Command code         |
| 2        | Target MUD| String  | Name of target MUD   |
| 3        | Target    | String  | Player to notify     |
| 4        | Source    | String  | Player that is paging|
| 5        | Source MUD| String  | Origin MUD           |
| 6        | Message   | String  | The message to relay |

### tell reply (2010)

Response to a tell or page. 

1. 2010 
2. The wizard to respond to 
3. The mud to send to 
4. "Server" dummy name 
5. The name of our mud 
6. The message, e.g.  
   %s is not currently on-line; 	
   %s is currently mailing or writing; 	
   Your message has been delivered 

### who request (2020)

 Request login information from another mud. 	 

| Position | Parameter | Type    | Description          |
| -------- | --------- | ------- | -------------------- |
| 1        | "2020"    | Integer | Command code         |
| 2        | Target MUD| String  | Name of target MUD   |
| 3        | Source    | String  | Player that is paging|
| 4        | Source MUD| String  | Origin MUD           |

### who reply (2030)

Respond to a who request

1. 2030 
2. The mud to respond to 
3. The player to respond to 
4. The name of our mud 
5. Information about an active player  [level] name title ...  (etc.) 

### page (2040)

Send a "page" to another player.

| Position | Parameter | Type    | Description          |
| -------- | --------- | ------- | -------------------- |
| 1        | "2040"    | Integer | Command code         |
| 2        | Target    | String  | Player to notify     |
| 3        | Target MUD| String  | Name of target MUD   |
| 4        | Source    | String  | Player that is paging|
| 5        | Source MUD| String  | Origin MUD           |
| 6        | Message   | String  | Optional message or "NONE" |

### mudinfo request (2050)

Request information on another mud 

| Position | Parameter | Type    | Description          |
| -------- | --------- | ------- | -------------------- |
| 1        | "2050"    | Integer | Command code         |
| 2        | MUDName   | String  | Name of queried MUD  |
| 3        | Player    | String  | Player who is asking |

### mudinfo reply (2055)

Send out information about ourselves. 	 

1. 2055 
2. Reserved for local communication 

### wiz (3000)

Say something over the intermud wizard channel. 	 

| Position | Parameter | Type    | Description          |
| -------- | --------- | ------- | -------------------- |
| 1        | "3000"    | Integer | Command code         |
| 2        | Source    | String  | Origin Player        |
| 3        | Source MUD| String  | Origin MUD           |
| 4        | Message   | String  | The message to relay |

### dns purge (4000)

Force a mudlist purge. 	 

| Position | Parameter | Type    | Description          |
| -------- | --------- | ------- | -------------------- |
| 1        | "4000"    | Integer | Command code         |
| 2        | Source    | String  | Origin Player        |

### reget (4020)

Request a mudlist from the bootmaster. 

| Position | Parameter | Type    | Description          |
| -------- | --------- | ------- | -------------------- |
| 1        | "4020"    | Integer | Command code         |
| 2        | Source    | String  | Origin Player        |

### stats (4040)

Get status information. 	 

| Position | Parameter | Type    | Description          |
| -------- | --------- | ------- | -------------------- |
| 1        | "4040"    | Integer | Command code         |
| 2        | Source    | String  | Origin Player        |

### mute (4060)

Mute/unmut a complete mud. 	 

| Position | Parameter | Type    | Description          |
| -------- | --------- | ------- | -------------------- |
| 1        | "4060"    | Integer | Command code         |
| 2        | MUDName   | String  | MUD to mute          |

### debug (4070)

Toggle intermud debugging on/off. 	 

| Position | Parameter | Type    | Description          |
| -------- | --------- | ------- | -------------------- |
| 1        | "4070"    | Integer | Command code         |
| 2        | PLayer    | String  | Player who issued the command         |

##  History
## Intro to Intermud 1.00
Stryker announced the final version 1.0 with the following words on a mailing list.

:::note
Intermud is much easier to install now. Your best bet would be to copy your src/ directory to another directory and run the patch that is detailed in the README. I would like to announce the release of Intermud v1.00. Intermud has been in beta for the last 2 years and I believe that now it is stable. I have updated the README.intermud file to include complete instructions on how to add Intermud to your mud. 
I've made it MUCH easier than it was. There are 3 ways you can go about it :
- 1) I have created a CircleMUD distribution with Intermud already built in, all you do is build it like you would normally build CircleMUD. You can retrieve that from ftp://styx.ph.msstate.edu/pub/tempus/intermud/circle30bpl11+imud.tar.gz or from ftp.circlemud.org after it's moved from the incoming directory. 
- 2) Most of you can get away with the patch distribution. This will patch in everything that you need. You can retrieve it from ftp://styx.ph.msstate.edu/pub/tempus/intermud/intermud_circle30bpl11.tar.gz or from ftp.circlemud.org after it's moved from the incoming directory. 
- 3) Last is the regular Intermud distribution. It contains all of the files and information that you need, however, you have to put it all together yourself, which is actually not that hard. You should choose this option only if the patch distribution above does not work for you. ftp://styx.ph.msstate.edu/pub/tempus/intermud/intermud-v1.00.tar.gz or from ftp.circlemud.org after it's moved from the incoming directory. 
I have attached the README.intermud file to this message. Hope to see you connected, Stryker@Tempus 
:::

:::note[Readme]
```
******************************************************************************* 
****************************** Intermud v1.00 ********************************* 
******************************************************************************* 
INTRODUCTION 
Well, it's been about 2 years now that Intermud has been in a BETA type form 
and now I think the major networking mechanics are pretty solid. In the past I 
have never included decent documentation or an easy way to get Intermud 
installed on a new mud. Hopefully with this release and this new README more 
people will be able to install Intermud and get it up and running. 
It's actually very simple. There are three ways I guess that you can get 
Intermud up and running. 
The first, and fastest would be to FTP circle30bpl11+imud.tar.gz, unpack it and 
type make all, presto, it's all but done. However, for the most part that's 
extreamly unlikely. Most will want to use the intermud_circle30bpl11.patch file 
which will patch in everything for you. And last but not least there will be 
some of you that have so heavily modified Circle 3.0 that the patch file won't 
work, or you are trying to port Intermud to a different mud platform. Either 
way, I have documented some instructions on all three senerios below. 

HOW IT WORKS 
There are three parts to the Intermud enviornment. The first is nothing that 
you have to run or worry about, I call it the BOOTMASTER. The BOOTMASTER runs 
at styx.ph.msstate.edu and all of the muds connected to the Intermud network 
connect to it to get the MUDLIST. The second part is the intermud_client.c 
source which compiles into ../bin/intermud. This is your client version of the 
software. Your mud talks to your local copy of the client which in turns send 
messages etc to the rest of the muds connected to the Intermud network. Your 
copy of intermud is a seperate binary executable that runs as it's own process 
on your machine. The third part is act.intermud.c which is compiled into your 
mud and acts as the interface to the Intermud process. 

HOW TO INSTALL IT 
If you have FTP'd circle30bpl11+imud.tar.gz then start here. 
1) Uncompress and Untar circle30bpl11+imud.tar.gz into a place where you want 
   to run it from. 
   gzip -dc circle30bpl11+imud.tar.gz | tar xvf 
   This will create a directory called circle30bpl11. You can then cd into 
   this directory and type ./configure (part of the regular CircleMUD install 
   process) 
2) Now, cd into the src/ directory and edit the intermud.h file, you will have 
   to change what you need, all of the settings are explained. 
3) Edit the intermud_script file and make sure that the port specified matches 
   the INTERMUD_PORT you dave defined in the intermud.h file. 
4) Type :- ``make all`` This will compile CircleMUD for you and Intermud as 
   well. You can now move on to the HOW TO GET IT TO RUN section. 
   
If you have FTP'd intermud_circle30bpl11_patch.tar.gz and want to patch 
Intermud into your copy of CircleMUD, start here. 
1) Move intermud_circle30bpl11_patch.tar.gz into the top level directory of 
   your mud. IE /usr/local/src/circle30bpl11 
2) Uncompress and Untar intermud_circle30bpl11.tar.gz. 
   ``gzip -dc intermud_circle30bpl11.tar.gz | tar xvf`` 
   This will create these files :- README.intermud intermud_circle30bpl11.patch 
   intermud_script 
3) Now you will patch Intermud into your source tree. Type in :
   ``patch -p0<intermud_circle30bpl11.patch`` 
   Watch for any errors, but there shouldn't really be any. 
4) Now, cd into the src/ directory and edit the intermud.h file, you will have 
   to change what you need, all of the settings are explained. 
5) Edit the intermud_script file and make sure that the port specified matches 
   the INTERMUD_PORT you dave defined in the intermud.h file. 
6) Type :- make all This will compile CircleMUD for you and Intermud as well. 
   You can now move on to the HOW TO GET IT TO RUN section. 
   
If you have FTP'd intermud_v1.0.tar.gz then you must figure that you have 
modified stock CircleMUD to the point of non-recognition. Which is fine, 
because if you've done this then you probably know what you are doing and you 
should be able to follow these instructions and wedge Intermud into your mud. 
1) Uncompress and Untar intermud-v1.0.tar.gz in your src/ directory and move 
   the intermud_script file up one directory to your top level mud directory. 
   2) Follow the instructions in intermud.doc to get the important intermud hooks into your comm.c file. 
   3) Add act.intermud.c to your Makefile. It must be compiled in with the rest of your mud's source. 
   4) Add the following commands to your interpreter.c :- interwho interwiz intertell interpage intermud mudinfo You'll have to add it in two places, First the ACMD(do_interwho) etc, and you'll have to add the commands to the cmd_info struct as well. 
   5) Edit the intermud.h file, you will have to change what you need, all of the settings are explained. 
   6) Compile the intermud_client.c file :- gcc -o ../bin/intermud intermud_client.c That should work perfectly under Linux. If you are using another O/S you will most probably have to add some libraries to the gcc call. For SunOS try adding -lnsl, for others try -lsocket. I would be interested in hearing from anyone who's got the correct libs for their OS so that I can add it to this document. 
   7) Edit the intermud_script file and make sure that the port specified matches the INTERMUD_PORT you dave defined in the intermud.h file. 
   8) You can now move on to the HOW TO RUN IT section. HOW TO RUN IT Now that you have edited the intermud.h file, compiled everything, and you have edited the intermud_script file to ensure that the PORT matches the INTERMUD_PORT in your intermud.h file you can simply type :- ./intermud_script & >From your top level mud directory. 
   
   Now you can start your mud as you normaly would. Commands you can use while in the mud :
   * interwiz : Will send a message out to the immortals on the other muds connected. 
   * interwho : Will get a who listing from another mud. 
   * intertell: Will send a tell to player on another mud. 
   * interpage: Will page player on another mud. 
   * mudlist : Will show all of the muds connected to the network 
   * mudinfo : Will retrieve some info from a mud. 
   * intermud : Controls the intermud client program. You can disconnect, connect, purge the mudlist, reget the mudlist, display some statistics etc etc. 
   
THE FUTURE 
Sometime in the future I plan to add additional functionality like intermail, 
interboard messages and anything else that I or you might think of. Stay tuned. 

IF YOU HAVE PROBLEMS 
If you have any problems getting it up and running, you can reach me at :
- caustin@pinc.com or telnet to :- styx.ph.msstate.edu PORT 2020 

And there is also an intermud mailing list, at :- intermud@dmv.com 

Hope you enjoy, Stryker@Tempus 

REVISION HISTORY 
0.00 to 0.51 
    - Never released to the public for beta testing. Progress was what you see in intermud today. 
0.52 
    - Added new command mudinfo. 
    - Made protocol changes to allow for additional services. 
0.53 
    - Fixed bug in mudinfo command that would crash a mud.
    - Reduced services from 32 bytes to an 8 byte bitmap. 
    - You will now see your mud in a mudlist and intermud stat command. 
0.54 
    - Fixed the nasty server bug that prevented every mud, after the first mud, not to connect to the server. 
    - Fixed what seems to be a Linux pre-ELF GCC compile bug in decode_services() 
    - Converted interlog() function from VARARGS to STDARG. 
    - Special thanks to Fireball@Tempus for a new home for the intermud server. 
0.55 
    - Revisions lost in a disk crash 
0.56 
    - Did huge cleanup on the code, moved a lot of things into seperate functions 
    - Drasticly cleaned up the log messages 
1.00 
    - More code clean up, some minor bug fixes 
    - NEW README.intermud file 
    - Created complete CircleMUD distribution with intermud built in. 
    - Created a circle30bpl11 patch
```
:::
