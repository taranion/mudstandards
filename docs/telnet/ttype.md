---
sidebar_label: TTYPE (24)
description: Can be used to determine or select the type of terminal emulator on the client.
---
# Terminal Type (TTYPE)

**Option code:** 24

**See also**: [RFC 1091](https://www.rfc-editor.org/rfc/rfc1091.html)

Originally by a server to query all supported types of terminal emulation and 
select one that matches the server best. Since ANSI terminals are omnipresent,
this telnet option is obsolete for its original purpose.

MUD servers keep the mechanism of the protocol, but instead of cycling through 
all supported terminal emulations, query the terminal type 3 times and assign 
different meanins to those values. See [MTTS](/mud/mtts)