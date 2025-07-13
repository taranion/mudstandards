---
sidebar_label: MMCP
---
# MUD Master Chat Protocol (MSSP)

**Source**: [Mudhalla](https://mudhalla.net/tintin/protocols/mmcp/)

MMCP is a decentralized chat protocol which allows MUD clients to communicate with each other over a TCP/IP connection.

## Establishing a Connection

**Caller:**

When a TCP/IP connection is made the caller sends a connection handshake which looks as following: ``CHAT:<chat name>\n<ip address><port>``. The sprintf syntax is: ``CHAT:%s\n%s%-5u``. The port must be 5 characters, padded on the right side with spaces. Once this string has been sent it waits for a response from the receiver. If a "NO" is received the call is cancelled. If the call was accepted the string ``YES:<chat name>\n`` is received.

**Receiver:**

When a socket call is detected it accepts the socket then waits for the ''CHAT:'' string to be sent from the caller. If the receiver wishes to deny the call, the string "NO" needs to be sent back to the caller. To accept the call, the string ``YES:<chat name>\n`` is sent back.
The quote characters mean that the encased words are a string, the quotes themselves should not be send.

## Chat Data Blocks
A chat data block looks as following: ``<COMMAND BYTE><data><END OF COMMAND>``. All chat data needs to follow this format except for the initial connection handshake and file transfer blocks which are a fixed size and don't need the ``<END OF COMMAND>`` byte.

## List of the &lt;COMMAND BYTE&gt; values
```
CHAT_NAME_CHANGE                      1
CHAT_REQUEST_CONNECTIONS              2
CHAT_CONNECTION_LIST                  3
CHAT_TEXT_EVERYBODY                   4
CHAT_TEXT_PERSONAL                    5
CHAT_TEXT_GROUP                       6
CHAT_MESSAGE                          7
CHAT_DO_NOT_DISTURB                   8

CHAT_VERSION                         19
CHAT_FILE_START                      20
CHAT_FILE_DENY                       21
CHAT_FILE_BLOCK_REQUEST              22
CHAT_FILE_BLOCK                      23
CHAT_FILE_END                        24
CHAT_FILE_CANCEL                     25
CHAT_PING_REQUEST                    26
CHAT_PING_RESPONSE                   27
CHAT_PEEK_CONNECTIONS                28
CHAT_PEEK_LIST                       29
CHAT_SNOOP_START                     30
CHAT_SNOOP_DATA                      31

CHAT_END_OF_COMMAND                 255
```

## MMCP Commands

``<CHAT_NAME_CHANGE><new name><CHAT_END_OF_COMMAND>``

When a user changes their chat name the new name needs to be broadcast to all of their connections.

``<CHAT_REQUEST_CONNECTIONS><CHAT_END_OF_COMMAND>``

Requests connections from another connection asking to see all the people that person has marked as public, then try to connect to all of those yourself.
``<CHAT_CONNECTION_LIST><address>,<port>,<address>,<port><CHAT_END_OF_COMMAND>``
The receiver needs to put all the IPs and port numbers of the public connections in a comma delimited string and send them back as a connection list.

``<CHAT_TEXT_EVERYBODY><chat message><CHAT_END_OF_COMMAND>``
Used to send a chat message to everybody. The chat message needs to be generated on the sender's side, including the line feeds and the ``<chat name>`` chats to everybody" string.

Receiver:

If the chat connection isn't being ignored, you simply print the string. If you have any connections marked as being served you need to echo this string to those connections. Or if this is coming from a connection being served, you need to echo to all your other connections. This allows people who cannot connect directly to each other to connect with a 3rd person who *can* connect to both and be a server for them.
``<CHAT_TEXT_PERSONAL><chat message><CHAT_END_OF_COMMAND>``
This works the same way as CHAT_TEXT_EVERYBODY. The text should be changed so the receiver knows this was a personal chat and not broadcast to everybody.

Receiver:

Just print the incoming message if you aren't ignoring this connection.
``<CHAT_TEXT_GROUP><group><chat message><CHAT_END_OF_COMMAND>``
The group name is a 15 character string. It must be 15 characters long and padded on the right with spaces to fill it out. The text should be changed so the receiver knows this was a group chat and not broadcast to everybody.

Receiver:

Just print the incoming message if you aren't ignoring this connection.
``<CHAT_MESSAGE><message><CHAT_END_OF_COMMAND>``

This should be used for generic system messages, like accepting or declining a chat connection, and marking someone public or private. To let the other side know the message is generated from the chat program it is a good idea to make the string resemble something like: ``\n<CHAT> %s has refused your connection because your name is too long.\n``

Receiver:

Just print the message string.
``<CHAT_VERSION><version string><CHAT_END_OF_COMMAND>``
The version string should include your mud client's name and version.

``<CHAT_FILE_START><filename,length><CHAT_END_OF_COMMAND>``
Used to send a chat connection a file. The filename should not include the path. Length is the size of the file in bytes.

Receiver:

First should check if you are allowing files from this connection. Make sure the filename is valid and that the length was transmitted. If for any reason the data isn't valid or you don't want to accept files from this person a CHAT_FILE_DENY should be sent back to abort the transfer. If you want to continue you need to start the transfer by requesting a block of data with CHAT_FILE_BLOCK_REQUEST.

``<CHAT_FILE_DENY><message><CHAT_END_OF_COMMAND>``
Used when a CHAT_FILE_START has been received and you want to prevent the transfer from continuing. ``<message>`` is a string telling the reason it was denied. For example, if the file already exists you might deny it with: "File already exists."

Receiver:

Print the deny message. Deal with cleaning up any files you opened when you began the transfer.
``<CHAT_FILE_BLOCK_REQUEST><CHAT_END_OF_COMMAND>``
Sent to request the next block of data in a transfer.

Receiver:

Create a file block to be sent back. File blocks are fixed length so you must not add the CHAT_END_OF_COMMAND byte. If the end of file is reached you need to send a CHAT_FILE_END, close the file, and let the user know the file transfer is completed.
``<CHAT_FILE_BLOCK><block of data>``
A file block is 500 bytes. No CHAT_END_OF_COMMAND should be added.

Receiver:

The receiver needs to keep track of the number of bytes written to properly write the last block of data. The last block is unlikely to be the full 500 bytes. File transfers are receiver driven, so for each block of data you accept, you need to send another CHAT_FILE_BLOCK_REQUEST back out to get more data.

``<CHAT_FILE_END><CHAT_END_OF_COMMAND>``
Close up your files and be done with it.

``<CHAT_FILE_CANCEL><CHAT_END_OF_COMMAND>``
Either side can send this command to abort a file transfer in progress.

``<CHAT_PING_REQUEST><timing data><CHAT_END_OF_COMMAND>``
The timing data is up to the ping requester.

``<CHAT_PING_RESPONSE><timing data><CHAT_END_OF_COMMAND>``
When receiving a CHAT_PING_REQUEST you should send back the timing data.

``<CHAT_PEEK_CONNECTIONS><CHAT_END_OF_COMMAND>``
The sender requests connections from another connection asking to see all the people that person has marked as public. The receiver needs to put all the ip addresses, port numbers, and names in a tilde delimited string and send them back as a peek list.

``<CHAT_PEEK_LIST><address>~<port>~<name>~<CHAT_END_OF_COMMAND>``
The receiver needs to put all the ip addresses, port numbers, and names in a tilde delimited string and send them back as a peek list.

``<CHAT_SNOOP_START><CHAT_END_OF_COMMAND>``
The sender requests to start or stop snooping data from a chat connection. The Receiver decides whether to allow snooping or not.

``<CHAT_SNOOP_DATA><message><CHAT_END_OF_COMMAND>``
Send by a mud client in snoop mode. The message should be echoed by the receiver, but not be further forwarded to avoid infinite loops.

## Links

If you want a link added, you can email me at mudclient@gmail.com.


