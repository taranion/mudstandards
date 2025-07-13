---
sidebar_label: MSLP
---
# MUD Server Link Protocol

**Source**: [Mudhalla](https://tintin.mudhalla.net/protocols/mslp/)

In 1998 the MXP (Mud eXtension Protocol) was developed by the author of zMUD which among other things allows the embedding of clickable links. The problem with MXP lies in that it contains a lot of other things and adds requirements that go far beyond the scope of something as simple as a clickable link.
MSLP seeks to exclusively address the task of handling clickable links.


# The MSLP Protocol

MSLP is negotiated by using the [MTTS](mtts) standard.

## Simple Links
To create the most basic link you need to insert text between the ``\e[4m`` and ``\e[24m`` codes. These are the VT100 codes to enable and disable underline. These exact codes must be used, so something like ``\e[32;4m``, ``\e[04m``, or any other random variant, would not be a valid start of a simple link. When clicked the client should send the command contained between the two tags to the server.

Since most clients will support underline, and there may be instances where you don't want to underline a link, you can use the ``\e[4;24m`` code as an alternative to start a simple link that is not underlined, you must still end the link using ``\e[24m``.

## Complex Links
If you want to create a complex link you need to use an **OSC (Operating System Command)** VT100 sequence.

The simple link stays the same, but it must be prefixed by the ``\e]68;1;;\a`` sequence. ``OSC 68;1`` supports two arguments, the first argument is a label, which can be used to describe what kind of link you are sending. Labels are case sensitive. The second argument is the command that is to be sent back to the server.
```
\e]68;1;SEND;say Hello World!\a\e[4m(click me)\e[24m
```
In the example above SEND is used as the label, which should be in all upper case letters. The text (click me) is printed, and when clicked the client should send 'say Hello World!' to the server.
If you send an unrecognized OSC code to a proper VT100 capable terminal the code will be ignored and it won't be displayed. Do make sure to add the \a code to terminate the OSC sequence. Not doing so is likely to result in odd behavior.

## Labels
As mentioned previously, labels are case sensitive. They serve to distinguish various types of links.

### SEND
The most commonly used label is likely to be the SEND label. The second argument is the command that is to be sent to the server.

### MENU
The MENU label is used to create a pop-up menu of links.
```
\e]68;1;MENU;{a tasty donut}{buy donut}{a loaf of bread}{buy bread}{a big tomato}{buy tomato}\a\e[4mshopping list\e[24m
```
The second argument of the MENU label is a list of key / value pairs separated by braces. The first item of the pair should contain the name, and the second item of the pair should contain the command.
Since each key/value pair exists of two items there is no need for commas, and subsequently they are not a requirement, nor allowed. If you however feel the need to distinguish key value pairs it is allowed to add one or more spaces as following.
```
\e]68;1;MENU;{a tasty donut}{buy donut} {a loaf of bread}{buy bread} {a big tomato}{buy tomato}\a\e[4mshopping list\e[24m
```
Including curly braces in the item name or command is not allowed, and the total number of opening braces must match the total number of closing braces for the link to be valid. The \ character is assumed to be literal and cannot be used for escaping. The \a and \0 sequence cannot be used. Other than that there are no real restrictions on what may be included.

## Secure Links
Secure links are created with the ``\e]68;2;;\a`` sequence. Secure links are reserved for internal use by the client.

## Jump Marks
Jumps marks are created with the ``\e]68;4;;\a`` sequence. The first argument is an optional label. The second argument should be a marker name.
## Jump Links
Jump links are created with the ``\e]68;5;;\a`` sequence. The first argument is an optional label. The second argument should be a marker name. When clicked the client should search the scrollback buffer for a matching Jump Mark and move to it.
