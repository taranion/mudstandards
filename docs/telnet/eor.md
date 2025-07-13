---
sidebar_label: EOR (25)
description: Signal the end of a prompt to the client
---
# End Of Record (EOR)

**Option code:** 25

**See also**: [RFC 855](https://www.rfc-editor.org/rfc/rfc855.html)

When enabled, this Telnet option allows sending the bytecode 239 (EOR) to 
signal the end of a data unit. In MUD environments it is used when sending a 
prompt (which does not end with a newline), but inform the client that it 
should display the content from the receive buffer to the user.  


| Tokens         | Bytes     | Meaning                                           |
| -------------- | --------- | ------------------------------------------------- |
| IAC WILL ECHO  | 255 251 25 | Server: I will/would like to send EOR codes      |
| IAC WON'T ECHO | 255 252 25 | Server: I won't send EOR codes                   |
| IAC DO ECHO    | 255 253 25 | Client: Okay to send EOR codes                   |
| IAC DON'T ECHO | 255 254 25 | Client: Don't/Stop sending EOR codes             |
