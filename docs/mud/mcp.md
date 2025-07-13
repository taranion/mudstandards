---
sidebar_label: MCP
---
# MUD Client Protocol (MCP)

**Source**: [The MUD Client Protocol (MCP)](https://www.moo.mud.org/mcp/)

## Introduction

Advances in MUD server design, particularly the development of in-server programming languages, coupled with the replacement of terminals and telnet sessions by personal computers and customized (and frequently user-programmable) clients have given rise to a demand for MUD-based applications    which make use of modern client capabilities such as windowing    systems, local file storage, and richer text display. At the same time, many MUD servers have retained the model of a single 7-bit ASCII channel per client, constraining the application    author's ability to design protocols.

The MUD Client Protocol (MCP) defines a simple and standard message format for the protocols used in constructing these applications. It is designed to be readily distinguished from the normal stream of MUD output and user commands and easily    parsed.

## Motivation and Goals

MCP is an attempt to provide a standard message format on which to build MUD-based client-server applications. As described above, many MUDs place restrictions on the user's I/O channel. Given these, a survey of MCP's goals is in order:      

- MCP is simple and open-ended. Rather than attempting to define the messages which implement applications built on MCP, it defines the format these messages should follow, and leaves the details of protocol message design to application authors.
- It is expressed in 7-bit ASCII, permitting it to be carried over channels with restricted character sets.
- Its messages have a distinctive format; they may be carried on the same channel with, and readily recognized and removed from, the stream of "in-band" MUD commands and output.
- MCP makes only minimal distinctions between clients and servers. In this respect, it is applicable both to traditional client-server communications and to server-to-server communications.
- MCP has little notion of state; instead, maintenance of state beyond the existence of a MCP-capable connection between the server and client is assumed to be the job of application-specific protocols.

## Information

- [The MCP 2.1 protocol specification](mcp2)
- ​[MCP-dev mailing list archive](http://www.research.att.com/~davek/mcp-dev/)
- [Simpleedit package specification](https://www.moo.mud.org/mcp/simpleedit.html): A simple local editing package
- [JHCore MCP 2.1 Implementation](http://www.awns.com/mcp/)
