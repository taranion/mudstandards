---
sidebar_label: loci.hotkey
---
# The ``Loci.Hotkey 2`` package

A GMCP module to program and add dynamic lables to the client hotkey menu.<br/>
If client claims support, server will send hotkey definition update messages asynchronosly, as they become available, at the server's discretion. 
Source: [LociTerm](https://www.last-outpost.com/LO/protocols/loci.hotkey.html)

### Hotkey definitions
| value | Key    |
| ----- | ------- |
| pgup  | Page Up |
| pgdn  | Page Down |
| home  | Home    |
| end   | End     |
| f1    | F1      |
| ...   | ...     |
| f20   | F20     | 

## Loci.HotKey.Get 
Request for server to send/resend all of the current hotkey definitions to the client.  
May be sent at any time, or not at all.  Intended use is to indicate to server 
that all previously sent updates may have been lost due a client reset, and 
need to be resent.  
````json
"Loci.Hotkey.Get" { }
````

## Loci.HotKey.Edit (2+)
**Since version 2**

Message from client to server to set a server side hotkey expansion.<br/>
Expectation is that if the server recieves the vt/xterm hotkey sequence, it 
will substitute in the provided macro text.  May be sent at any time, when
the client wishes to inform the server of a definition or change, such as
when user edits a hotkey with the client UI.
````json
"Loci.Hotkey.Edit" {         
    name: "<keyname>",       
    label: "<display label>"  
    macro: "alphanumeric string to set"
    sends: "seq" or "macro" 
}
````
| Key   | Required | Value       | Description                                                  |
| ----- | -------- | ----------- | :----------------------------------------------------------- |
| name  | Yes      | String      | A valid identifier (see above) for the key to define     |
| label | No       | String      | Name to appear in on-screen button                       |
| macro | No       | String      | The string to send when the key is pressed               |
| sends | No       | "seq","macro"| "seq": Client should send VT or XTerm function key sequence<br/>"macro": Client send conent ``macro`` parameter |

## Loci.Hotkey.Set
Message from server to client containing a hotkey definition.  May be sent
at any time.  name: is a required field.  label: is a UI display name,
intended to be used in on-screen buttons.
````json
"Loci.Hotkey.Set" {         
    name: "<keyname>",       
    label: "<display label>"  
    macro: "alphanumeric string to set"
    sends: "seq" or "macro" 
}
````
| Key   | Required | Value       | Description                                                  |
| ----- | -------- | ----------- | :----------------------------------------------------------- |
| name  | Yes      | String      | A valid identifier (see above) for the key to define     |
| label | No       | String      | Name to appear in on-screen button                       |
| macro | No       | String      | The string to send when the key is pressed               |
| sends | No       | "seq","macro"| "seq": Client should send VT or XTerm function key sequence<br/>"macro": Client send conent ``macro`` parameter |

The macro: string should always be sent with
    at least one terminating '\n' newline character, whether the macro string
    ends in a \n or not.  Note that if the macro: string isn't defined, the
    client should send the vt/xterm function key sequence instead of sending an
    empty line.

## Loci.Hotkey.Reset
Message from server to client indicating that the hotkey definintion MUST
be reset to the client default value.  if name: not present, ALL
definitions MUST be reset to defaults.

````json
"Loci.Hotkey.Reset" {
    name: "<keyname>"   // optional
}
````
