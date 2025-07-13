---
sidebar_label: ANSI
description: Control codes
---
# ANSI / ECMA-48

**Source**: [ECMA International](https://ecma-international.org/publications-and-standards/standards/ecma-48/)

The ANSI/ECMA document defines a standard for in-band signaling of control codes (e.g. for color, cursor location, font styling, tab stops ...)

Terminal vendors like DEC usually extended the ANSI standard by custom control codes. From those terminal vendors, the control codes of DEC (with their VT100+ series) are widely adopted.

## ANSI is a two-way street
In theory ANSI works in two ways - you can *send* data to the remove party, but you can also *request* reports from the remoty party. Such reports are useful e.g. for determining the cursor location or querying the terminal for capabilities.

Since most MUD servers only use ANSI to change color or font styling, most MUD clients just implemented that subset of the standard and did not bother with a full terminal emulation.

## Just ANSI styling or full terminal emulation?
A way for clients to express their capabilities is setting the ANSI and VT100 [MTTS](../mud/mtts#mud-terminal-type-standard-1) flags.

If the client does not support MTTS, the server could always try to simply send the requests and check if reports are returned. To do that, 
- the client needs to be put in a *character-at-a-time mode* (otherwise responses get buffered) and 
- the *local echo* needs to be disabled (otherwise the returned values appear on screen and confuse players)