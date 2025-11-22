---
sidebar_label: mudstandards.window
---
# The ``mudstandards.window`` package

## mudstandards.window.open

|                           |             |
| ------------------------- | ----------- |
| **Type**                  | `combining` |
| **Required**              | No          |
| **Additional properties** | Not allowed |

**Description:** This message opens a new frame/window/dock in the client

| Property                   | Type             | Title/Description                                                                                                                                                                                |
| -------------------------- | ---------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| + [id](#id )               | string           | **Mandatory**: A unique identifier for the room.                                                                                                                                                 |
| - [title](#title )         | string           | **Optional**: The name to appear above the window. Without it, no space should be reserved                                                                                                       |
| + [type](#type )           | enum (of string) | **Mandatory**: How will this frame be handled: An external (moveable) window, a docked frame or a  child frame of another window or a tab, that can be displayed alternatively to the 'parent'   |
| - [parent](#parent )       | string           | **Optional**: For windows of type 'child', the parent frame they belong to. For windows of type 'tab', the reference frame to alternate with.                                                    |
| + [align](#align )         | enum (of string) | **Conditional**: On which side of the reference window shall this window appear                                                                                                                  |
| + [sizeValue](#sizeValue ) | integer          | **Conditional**: Size of the new window. Consult 'sizeUnit' for meaning                                                                                                                          |
| - [sizeUnit](#sizeUnit )   | enum (of string) | **Optional**: Is the size to be interpreted as characters(c), pixel(px) or percent(%)                                                                                                            |
| - [content](#content )     | enum (of string) | **Optional**: What type of content should be displayed in that frame? <br />\`content="terminal"\` is another terminal emulator, while \`content="webview"\` is an HTML webpage with Javascript. |
| - [url](#url )             | url              | **Mandatory**: In case of a \`webview\` content, this contains the URL to open                                                                                                                   |

#### <a name="id"></a>1. Property `id`

|              |          |
| ------------ | -------- |
| **Type**     | `string` |
| **Required** | Yes      |

**Description:** **Mandatory**: A unique identifier for the room.

### <a name="title"></a>2. Property `title`

|              |          |
| ------------ | -------- |
| **Type**     | `string` |
| **Required** | No       |

**Description:** **Optional**: The name to appear above the window. Without it, no space should be reserved

### <a name="type"></a>3. Property `type`

|              |                    |
| ------------ | ------------------ |
| **Type**     | `enum (of string)` |
| **Required** | Yes                |

**Description:** **Mandatory**: How will this frame be handled: An external (moveable) window, a docked frame or a  child frame of another window or a tab, that can be displayed alternatively to the 'parent'

Must be one of:
* "external"
* "docked"
* "child"
* "tab"

### <a name="parent"></a>4. Property `parent`

|              |          |
| ------------ | -------- |
| **Type**     | `string` |
| **Required** | No       |

**Description:** **Optional**: For windows of type 'child', the parent frame they belong to. For windows of type 'tab', the reference frame to alternate with.

### <a name="align"></a>5. Property `align`

|              |                    |
| ------------ | ------------------ |
| **Type**     | `enum (of string)` |
| **Required** | Yes                |

**Description:** **Conditional**: On which side of the reference window shall this window appear

Must be one of:
* "top"
* "bottom"
* "left"
* "right"

### <a name="sizeValue"></a>6. Property `sizeValue`

|              |           |
| ------------ | --------- |
| **Type**     | `integer` |
| **Required** | Yes       |

**Description:** **Conditional**: Size of the new window. Consult 'sizeUnit' for meaning

| Restrictions |        |
| ------------ | ------ |
| **Minimum**  | &ge; 1 |

### <a name="sizeUnit"></a>7. Property `sizeUnit`

|              |                    |
| ------------ | ------------------ |
| **Type**     | `enum (of string)` |
| **Required** | No                 |
| **Default**  | `"px"`             |

**Description:** **Optional**: Is the size to be interpreted as characters(c), pixel(px) or percent(%)

Must be one of:
* "c"
* "px"
* "%"

### <a name="content"></a>8. Property `content`

|              |                    |
| ------------ | ------------------ |
| **Type**     | `enum (of string)` |
| **Required** | No                 |
| **Default**  | `"terminal"`       |

**Description:** **Optional**: What type of content should be displayed in that frame? 
`content="terminal"` is another terminal emulator, while `content="webview"` is an HTML webpage with Javascript.

Must be one of:
* "terminal"
* "webview"

### <a name="url"></a>9. Property `url`

|              |       |
| ------------ | ----- |
| **Type**     | `url` |
| **Required** | No    |

**Description:** **Mandatory**: In case of a `webview` content, this contains the URL to open


## mudstandards.window.close (TODO)


