---
sidebar_label: ZMP (Option 93)
---
# Zenith Client Protocol (ZMP)

**Source**: [Discworld](https://discworld.starturtle.net/external/protocols/zmp.html)

## Preamble

This document is a technical specification defining an extension to TELNET, defined in [RFC854](http://www.faqs.org/rfcs/rfc854.html).

Advanced MUD features  are in need of a protocol for sending messages between the client and  server which are not part of the player viewed content.	Several current  protocols exist for this purpose, but each have flaws.	A new solution is proposed to solve these problems.

The solution, ZMP,  makes use of the existing TELNET capabilities to their fullest extent,  simplifying implementation and ensuring maximum flexibility.

Please feel free to contact the author(s) of this document, and the [ZMP specification](http://zmp.sourcemud.org/spec.shtml#spec) with comments or suggestions.	Keep the unconstructive flames and unrelated questions to yourself, please.

In the specification, several formatting styles are used.  *must* and *must not* indicate required behavior for compliance.  *should* and *should not* indicate suggested but non-required behavior for compliance.  Words formatted as IAC represent bytes with a well-defined value a specified in [RFC854](http://www.faqs.org/rfcs/rfc854.html).  Text formatted as user data represents a series of US-ASCII encoded bytes, excluding the quotation marks.



## ZMP Specification

ZMP is an extension building on [RFC854](http://www.faqs.org/rfcs/rfc854.html). The actual messages are sent using the TELNET subnegotiation feature.	 This helps to meet the requirement for simplicity; the TELNET parsing  and handling code should already be in most every server and client.	 Additionally, the safety requirement is handled here-in, as the IAC  escaping should already be handled.	As we're using the existing TELNET  layer, which serves as the client/server connection, ZMP also satisfies  the resource requirement.

The ZMP numeric TELNET code in decimal is *93*.  In hexadecimal it is 0x5D.

ZMP activation is handled using telnet negotiation.	The server *must* begin negotiation, by sending IAC WILL ZMP.	If a client supports ZMP, it *must* respond with IAC DO ZMP; otherwise, the client *should* send IAC DONT ZMP.

Once ZMP is successfully negotiated between the server and client,  bidirectional ZMP support is enabled for both client and server.	Once  enabled, ZMP may not be disabled.  The client *must not* begin ZMP negotiation; it may only respond to a server negotiation request.

A ZMP session corresponds to a telnet connection.	Once ZMP is  enabled, the ZMP session has begun, and persists until the telnet  connection is closed.

A command is comprised of a command name and a list of parameters.	 Arguments are free-form; they have no type associated, and no structure  associated.	They are similar to the argv[] array found in the main()  function in C/C++ programming.

Commands *must* be sent using a telnet subnegotiation with the ZMP telnet code.  Implementations *must* be prepared to accept at least 16KiB (16384 bytes) of data per subnegotiation.  Implementations *should not* attempt to send more than 16KiB of data per subnegotiation.

The data portion of the subnegotiation carries the command and parameters.	The command *must* be sent first.	The command *must* be followed by a NUL byte.	(A byte of value 0.)

ZMP command names *must* be comprised of only ASCII alpha-numeric characters (letters and numbers), dots (.), and dashes(-).	No other characters may be used.	Commands *must not* begin or end with a dot (.).

Zero or more parameters *may* follow.	An parameter *must not* contain a NUL byte, but may be comprised of any other byte values.	Also note that as per the telnet standard, any IAC bytes *must* be escaped using the double IAC sequence.	Each parameter *must* be followed by a single NUL byte.	In most cases, parameters will be simple ASCII (or UTF-8) strings.  Implementations *must* be prepared to accept an arbitrary number of parameters.

An example command sequence may look like: IAC SB ZMP my-command NUL parameter 1 NUL second parameter NUL IAC SE.

Commands may be sent from the server to the client, or the client to the server.

Commands *must* be sent in-order with other commands and the  regular MUD data stream.	For example, if the normal MUD data stream is  placed in a buffer, ZMP commands must also be placed in the same buffer. If the sending end sends MUD data, a ZMP command, and then more MUD  data, the receiving end must receive the data and ZMP command in that  same order.	This is required to allow ZMP to function in a markup style  of operation.

In order to increase interoperability between the multitude of MUD servers and clients available, software implementing ZMP *should* make use of [command packages](https://discworld.starturtle.net/external/protocols/zmp.html#packages), especially the [core package](https://discworld.starturtle.net/external/protocols/zmp.html#core).



## Core Commands

There are several basic commands that most software using ZMP will want or need.

Software does not need to support any or all of these commands in  order to use ZMP, or to be compliant with the ZMP specification.	 However, it is generally a very good idea to implement these.	If nothing else, they will serve well to help test an implementation, and also  function as learnng exercise on how to write custom [packages](https://discworld.starturtle.net/external/protocols/zmp.html#packages) and commands.

### zmp.ping

The zmp.ping command is a simple testing command.	This command uses no parameters.	When the zmp.ping command is received, the zmp.time command *must* be sent in response.

### zmp.time

This command sends a single parameter: the date and time.	The date and time used *must* be in UTC.	The format *must* be YYYY-MM-SS HH:MM:SS.	Example: 2003-07-02 12:20:34.	(That's July 2nd  of 2003, at 20 minutes and 34 seconds past hour 16, or 4pm.)

### zmp.ident

The zmp.ident command communicates the name and version of the software in use.

This command uses three parameters: (1) the name of the software  (i.e., "MUD Client"), (2) the version of the software (i.e., "2.5  Beta"), and (3) a short and concise description of the software (i.e. "A graphical MUD client for Whiz OS.").

The zmp.ident command *should* be sent by both the client and the server once the ZMP session begins.	The command *must not* be sent more than once per ZMP session.	In particular, reception of the zmp.ident does not require any response command.

The zmp.ident command data is intended for branding and debugging purposes.	Do not rely on the ident data to  determine features supported by the softare.	Instead, use the zmp.check command to check for required commands.

### zmp.check

This command is used to check for support for a particular [package](https://discworld.starturtle.net/external/protocols/zmp.html#packages) or command on the other end.	For example, if a server wishes to see if a connected client supports the org.sourcemud package, it would send a zmp.check command with the single parameter, "org.sourcemud.".	For a package name, there *must* be a . (dot) appended.	For commands, there *must not* be dot appended; simply use the command name.

When the zmp.check command is received, one of zmp.support or zmp.no-support *must* be sent back.	If the asked for command or package is indeed supported, send an zmp.support command with the package/command name as the sole parameter.	Otherwise, send an zmp.no-support command in return, with the package/command name in question as the sole parameter.

If the parameter of the zmp.check command ends with dot, the request is then for a package.	The affirmative response must be sent if *any* command in the package or a sub-package is available.	If a specific  command is required, the command should be explicitly queried, not jus  tits package.

The available commands and packages *must not* change after the ZMP session has begun.	In particular, if a zmp.check command queries the presence of a command or package, and an  affirmative response is sent in reply, the command or package may not be removed from the session.

The zmp.check command *should* only be sent once per ZMP session per parameter.	That is, if the support of the org.sourcemud package is queried once, it will not appear or disappear later in the ZMP session, so it should not be queried for again.

### zmp.support

This command is used only as a response to the zmp.check command.

### zmp.no-support

This command is used only as a response to the zmp.check command.

### zmp.input

The zmp.input command is to be sent from  the client to the server only.	The payload of this command is a single  parameter, which is to be interpreted by the server as a user command,  just as if the client had sent a normal line of text to the server.

The advantage of this command is that it allows embedded newline  characters, which may be useful when sending blocks pre-formatted text.



## Packages

The [ZMP specification](https://discworld.starturtle.net/external/protocols/zmp.html#spec) will help to build software that is capable of sending and receiving  commands.	However, this is of little use if the software does not  understand the messages sent to it; a custom protocol may as well be  used in that case.

Some existing protocols define the set of commands they support in  the specification itself.	While this guarantees that compliant software  will be able to communicate meaningful messages, it can reduce the  flexibility of the software.	Both MXP and MCP allow custom messages to  be defined.	ZMP also allows this to be done.

The package system for ZMP is similar to the idea of namespaces found in popular programming languages, such as C++ and Java.	Every command  sent over ZMP should belong to a package.	A package is simply a name,  and a specification of the commands that are available in the package.

When a command is sent, it should be in the form package.command.	For example, if sending a command named foo, in the package named bar, use the command name bar.foo.	Packages may be broken down further; for example, a package may be named org.sourcemud.

Custom packages *should* be named based on a domain controlled by the author.	For example, custom [Source MUD](http://sourcemud.org/) commands should be in the org.sourcemud package, because the domain for Source MUD is sourcemud.org.	Note that  the domain components are reversed.	This is very similar to the  recommended way of naming Java namespaces.

Software *should not* add custom commands to packages published by other parties.	I.e., if there is a package popular.mud defined by a third party, software *should not* add a new command to popular.mud.	Custom commands should be defined in your.domain.

If there is no unique domain associated with the software, simply use a meaningful name related to the software, with x- prepended to that  name.	For example, if software called MUD Server requires a custom  package, and MUD Server has no domain, the package name x-mud-server should be used.	Any packages published for use by third parties *should* be in a domain-identified package.	The x- package names should only be  used internally by a project, such as between a server/client pair  specifically written for each other.

The ZMP authors are eager to help the community define some standard  packages.	With more available standard packages, there can be more  functionality across disparate MUD servers and clients.	Standard  packages are expected to be added for common MUD needs and codebases.



## Document History

### draft-2009-03-18

- Expand preamble with slightly more desciptive text.
- Clarify ZMP code 93 is decimal, note that it is 0x5D in hexadecimal.
- Improve clarity of the example ZMP command wire transmission.
- Clarify that the server always negotiates and doing so enables two-way transmission.

### draft-2009-03-14

- Replace the terminology 'argument' with 'parameter'.
- Replace the terminology 'sub-request' with 'subnegotiation'.
- Replace references to AweMUD and awemud.net with Source MUD and sourcemud.org.
- Note that implementations must be prepared to accept an arbitrary number of parameters.
- Note that implementations must be prepared to accept at least 16KiB of data in a TELNET sub-negotiation for ZMP.

### draft-2006-10-07

- Split the non-core packages out of the core specification.

### draft-2005-09-22

- Added definition of a ZMP session.
- Clarified use and behavior of the zmp.ident command.
- Removed use of "you" and "your" pronoun.
- Clarified significance and behavior of zmp.check and its responses.

### draft-2005-01-28

- Added sound package.

### draft-2005-01-16

- Fixed typo in license description so the implementation clarification says what it should.
- Fixed typo of time.

### draft-2005-01-10

- Added a Creative Commons license to this document.
- Corrected misspelling of NUL.
- Added a color code definition section the color package, following  the same rules used in CSS1, and updated the color package to reference  the color code specification.
- Allow color.background to set any background color using a valid color code.
- Allow server to send color.define with an identifier of zero in order to change the default text color.

### draft-2004-07-06

- Update color.define spec for empty color codes.

### draft-2004-07-03

- Added initial color optional package spec.

### draft-2003-04-06

- Added the zmp.input command.

### draft-2003-10-07

- Removed use of the OOB term, its meaning is a bit different than I meant it.
- Reworded specification to be clearer.

### draft-2003-08-09

- Specified ZMP requests must be in-order with regular data-stream.

### draft-2003-07-08

- Moved the usage notes, implementation notes, telnet primer, and current technology notes to a separate Notes document.
- Heavily modified the preamble.
- Reformatted and modified the actual specification section.

### draft-2003-07-03

- HTML prettification.
- Updated the current protocols section.
- Fixed code sample bugs.
- Added example zmp.check implementation.
- Moved command handlers part of implementation notes to end of the section.

### draft-2003-07-02

- Fixed typo in send_zmp_command example code.
- Renamed all the example commands to be in zmp_* form.
- Added example implementation notes for managing and invoking commands.
- Moved implementation notes to end of document.
- Changed zmp.time command again, RFC822 is too hard to parse; using YYYY-MM-DD HH:MM:SS in UTC format instead.
- Added example zmp.ping implementation.

### draft-2003-07-01b

- After enough complaints, renamed SMP.	Now ZMP: Zenith MUD Protocol.

### draft-2003-07-01

- Moved core commands to the smp package.	It's just more correct that way.
- Fixed formatting bug in telnet sub-request section.
- The smp.time command was redefined to return the time in RFC 822 format.
- Added the usage notes section.

### draft-2003-06-21

- Added MUD protocol ACMP notes to current technologies section.

### draft-2003-06-15

- Got rid of smp.core package, those commands are now packageless (easier).
- Other minor cleanups.

### draft-2003-06-14

- Spelling fixes from aspell.
- Cleaned up sample code.	Fixed some bugs.
- Added packages section.
- First pass at an smp.core package specification.
- Made history section look a little nicer.
- Expanded preamble.

### draft-2003-06-13

- Fix a bug in the send_wrapper() code.

### draft-2003-06-12b

- Grammar/spelling/formatting fixes.
- Code cleanups and improvements for implementation notes.
- Improved HTML/CSS code.

### draft-2003-06-12

- Initial writing.



## License and Copying

This document is the copyrighted work of Sean Middleditch

Sean Middleditch offers limited rights to recipients of this document to distribute unmodified copies of the document in electronic form,  print form, or any other medium.	Implementations of the protocol  described in this document are wholly owned by their creators, and the  implementations are *not* derivative works of this document and they *not* subject to the terms of this license.

This work is licensed under a [Creative Commons License](http://creativecommons.org/licenses/by-nd/2.0/).
