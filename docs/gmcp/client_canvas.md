---
sidebar_label: client.canvas (draft)
---

# The `Client.Canvas` package

:::caution Draft proposal

This document is a starting point for discussion, not an established standard. Package names,
message names, and object shapes may change before version 1 is accepted.

:::

`Client.Canvas` lets a MUD server present a small, rich view as declarative JSON. The server sends an
ordered scene of shapes; the client renders it with its native UI toolkit. No Lua, JavaScript, HTML,
WebAssembly, or other server-supplied program is downloaded or executed.

Version 1 is a canvas, not a general widget tree. It is suitable for status panels, diagrams, maps,
portraits, and command links. Native controls, forms, and a general application model are outside its
scope.

## Design principles

- **Latest valid snapshot wins.** Each `Set` is a complete canvas. There are no revisions, patches,
  acknowledgements, or synchronized client/server state machines.
- **Version 1 is one feature set.** A client advertising version 1 implements the complete shape and
  link grammar.
- **Placement is a hint.** The client and user retain control of size, position, visibility, and
  whether remote images or external links are allowed.
- **Interaction uses links.** Drawing records carry OSC 8-inspired links with established `send:` and
  `prompt:` action schemes.
- **Motion has a static fallback.** Any shape may carry an `animate` object. A client that cannot or
  should not display motion ignores `animate` and draws the static shape.
- **Live values are lookups, not expressions.** A property may bind directly to retained GMCP data.
  Bindings repaint automatically but are simple scalar values. They cannot contain
  expressions or code.

Smudgy originated this protocol and is its reference implementation. The generic `Client.Canvas`
namespace lets other clients support it without adopting Smudgy's scripting API or implementation
language. `Client.Canvas` is the proposed canonical capitalization; GMCP package matching remains
case-insensitive.

The drawing model sticks to operations that map to GPU/GL renderers, browser Canvas 2D and `Path2D`,
and Qt's `QPainter` and `QPainterPath`. These are examples, not dependencies. A renderer can normalize
or tessellate paths and cache geometry or textures. A browser implementation may add DOM elements for
accessibility. A Qt implementation must select the nonzero fill rule because `QPainterPath` defaults
to odd-even. Antialiasing, font rasterization, and exact text metrics will vary by backend.

## Negotiating support

A client advertises version 1 through the normal GMCP mechanism:

```json
Core.Supports.Set ["Client.Canvas 1", "mudstd.frame 1"]
```
Support for GMCP-defined frames is optional and negotiated independently through `mudstd.frame`.

## Canvas lifecycle

### `Client.Canvas.Set`

Sent by the server to create or replace a named canvas.

```json
Client.Canvas.Set {
  "id": "vitals",
  "label": "Character vitals",
  "anchor": "top-right",
  "canvas": {
    "width": 320,
    "height": 160,
    "view_box": [0, 0, 320, 160],
    "fit": "contain",
    "scene": [
      {"kind": "rect", "width": 320, "height": 160, "rx": 8, "fill": "#151922ee"},
      {"kind": "image", "src": "https://game.example/portraits/ada.png",
       "alt": "Portrait of Ada", "x": 16, "y": 16, "width": 72, "height": 72,
       "fit": "cover", "border_radius": 8, "rotation": 0, "opacity": 1},
      {"kind": "text", "x": 104, "y": 18, "text": "Health", "size": 16},
      {"kind": "rect", "x": 104, "y": 46, "width": 200, "height": 18, "fill": "#303642"},
      {"kind": "rect", "x": 104, "y": 46, "width": 165, "height": 18, "fill": "#d14545"},
      {"kind": "text", "x": 204, "y": 55,
        "text": {"$bind": "Char.Vitals", "path": "/hp",
                 "format": "{} HP", "fallback": "--"}, "size": 13,
       "align_x": "center", "align_y": "center"},
      {
        "kind": "group",
        "link": {"href": "send:drink health", "id": "drink-health",
                 "label": "Drink a health potion", "hint": "Consumes one potion"},
        "children": [
          {"kind": "rect", "x": 210, "y": 112, "width": 94, "height": 32,
           "rx": 6, "fill": "#803838"},
          {"kind": "text", "x": 257, "y": 128, "text": "Drink",
           "align_x": "center", "align_y": "center"}
        ]
      }
    ]
  }
}
```

| Property | Required | Description |
| --- | --- | --- |
| `id` | Yes | Canvas identifier, unique for the current connection. Maximum 128 UTF-8 bytes. |
| `target` | No | Identifier of an existing canvas-content `mudstd.frame`. When omitted, the canvas is placed in the primary client view. |
| `label` | Yes | Short human-readable name for client UI and accessibility tools. |
| `anchor` | No | Primary-view placement hint. Ignored when `target` names a frame. |
| `canvas` | Yes | The portable canvas object described below. |

`anchor` is one of `top-left`, `top`, `top-right`, `left`, `center`, `right`, `bottom-left`,
`bottom`, or `bottom-right`. It defaults to `top-right`. The client may ignore it or present the
canvas in another native location.

For a given `id`, the last valid `Set` replaces the previous canvas in full. A client may
coalesce queued `Set` messages and render only the newest one. A malformed or over-limit `Set` is
ignored and the last valid canvas remains visible.

A `Set` updates content but does not override a user's local decision to hide or move that canvas.

### `Client.Canvas.Remove`

Sent by the server to remove a canvas:

```json
Client.Canvas.Remove {"id": "vitals"}
```

Removing an unknown `id` has no effect. A later valid `Set` with the same `id` creates it again,
subject to the user's local visibility preference. Removing a canvas also clears its private
`Client.Canvas.Data`. All canvases and their data are removed when the connection ends.

## Targets and `mudstd.frame`

When `target` is omitted, every version 1 client places the canvas in its primary view for that
session. The canvas may overlay that view, sit beside it, or be exposed through another native
layout. It must not cover the prompt or trusted client controls in a way the user cannot undo.

This repository's draft [`mudstd.frame`](./mudstandards_frame.md) package defines external, docked,
floating, child, and tab frames. A client supporting both packages includes `canvas` in
`mudstd.frame.support.content`. The server opens the frame before targeting it:

```json
mudstd.frame.open {
  "id": "vitals-frame",
  "type": "docked",
  "content": "canvas",
  "align": "right",
  "label": "Vitals",
  "sizeValue": 320,
  "sizeUnit": "px"
}
```

```json
Client.Canvas.Set {
  "id": "vitals",
  "target": "vitals-frame",
  "label": "Character vitals",
  "canvas": {
    "width": "fill",
    "height": "fill",
    "view_box": [0, 0, 320, 160],
    "fit": "contain",
    "scene": []
  }
}
```

When `mudstd.frame` is not advertised, a server must omit `target`. If `target` is supplied, it must name
an existing frame whose content type is `canvas`; otherwise the `Set` is ignored. Frame placement and
sizing belong to `mudstd.frame`; `Client.Canvas` clips its output to the frame bounds.

Closing a target frame removes each canvas currently targeting it, including its private data, as if
the server had sent `Client.Canvas.Remove`. Reopening a frame with the same identifier does not
resurrect old content; the server sends a new `Set`. Clients that do not implement `mudstd.frame` need no
frame model at all.

## The canvas object

The `canvas` sub-object contains layout information and the canvas's scene objects.

```json
{
  "width": 320,
  "height": 160,
  "view_box": [0, 0, 320, 160],
  "fit": "contain",
  "scene": []
}
```

| Property | Required | Description |
| --- | --- | --- |
| `width` | No | Positive logical-pixel number or `fill`. Defaults to `fill`. |
| `height` | No | Positive logical-pixel number or `fill`. Defaults to `fill`. |
| `view_box` | Yes | `[x, y, width, height]` scene rectangle mapped to the canvas bounds. |
| `fit` | No | `contain` preserves aspect ratio; `fill` stretches both axes. Defaults to `contain`. |
| `scene` | Yes | Drawing records in back-to-front paint order. |

Numeric `width` and `height` are preferred logical-pixel sizes, not physical display pixels. A client
may constrain them to the space available. The last two `view_box` values must be positive. With
`contain`, the `view_box` is uniformly scaled and centered, leaving transparent margins when the
aspect ratios differ. With `fill`, it is independently scaled on each axis to exactly fill the canvas.

The scene coordinate system has its origin at the upper left, with positive x to the right and
positive y downward. Coordinates and dimensions are finite JSON numbers in scene units. Clients may
convert them to IEEE-754 binary32; a value that overflows to infinity is invalid. Negative dimensions
and radii are treated as zero. Drawing, including strokes and `contain` margins, is clipped to the
canvas bounds.

## Drawing records

Every record has a `kind`. Records draw in array order, so a later record appears over an earlier
one. A group draws its children in their array order at its position in the parent scene.

All records accept these common properties:

| Property | Required | Description |
| --- | --- | --- |
| `kind` | Yes | Shape discriminator. |
| `id` | No | Preferred stable animation identity within this canvas. Maximum 128 UTF-8 bytes. |
| `opacity` | No | Number from 0 through 1. Defaults to 1. |
| `animate` | No | Per-property tweens, described under [Animations](#animations). |
| `transition` | No | Smoothing for changing properties, described under [Transitions](#transitions). |
| `transient` | No | With `animate`, stop drawing after every tween finishes. Defaults to `false`. |
| `link` | No | Link activated from this record's geometry. |

Group opacity and transforms affect all descendants. A child's opacity multiplies its inherited
opacity.

### Paint and stroke

A solid paint is a CSS color string. Version 1 clients must support `#RRGGBB` and `#RRGGBBAA` and
may support additional CSS color forms.

A stroke has this form:

```json
{"color": "#ffffff", "width": 2}
```

`color` defaults to black and `width` defaults to 1 scene unit. A missing `fill` means no fill and a
missing `stroke` means no stroke. Strokes are centered on the shape boundary and use butt caps and
miter joins with a miter limit of 4.

### `rect`

```json
{"kind": "rect", "x": 10, "y": 10, "width": 100, "height": 40, "rx": 6,
 "fill": "#336699", "stroke": {"color": "#ffffff", "width": 2}}
```

`x`, `y`, and `rx` default to 0. `width` and `height` default to 0. `rx` is a uniform corner radius,
clamped to half the smaller of `width` and `height`.

### `circle`

```json
{"kind": "circle", "cx": 50, "cy": 50, "r": 25, "fill": "#44aa66"}
```

`cx`, `cy`, and `r` default to 0.

### `line`

```json
{"kind": "line", "x1": 0, "y1": 0, "x2": 100, "y2": 100,
 "stroke": {"color": "#ffffff"}}
```

All coordinates default to 0. A line has no fill.

### `polyline` and `polygon`

```json
{"kind": "polyline", "points": [[0, 0], [20, 10], [40, 0]],
 "stroke": {"color": "#ffffff"}}
```

```json
{"kind": "polygon", "points": [[0, 0], [40, 0], [20, 30]], "fill": "#ffaa00"}
```

`points` is required. A polygon closes the last point to the first; a polyline remains open.

### `path`

```json
{
  "kind": "path",
  "d": "M 12 2 L 15 9 L 22 9 L 17 14 L 19 22 L 12 17 L 5 22 L 7 14 L 2 9 L 9 9 Z",
  "fill": "#ffd45c",
  "stroke": {"color": "#7a5b00", "width": 1.5}
}
```

`d` is required and contains SVG path data, not an SVG document. Version 1 supports every SVG path
command in absolute and relative form:

- `M`/`m` move
- `L`/`l`, `H`/`h`, and `V`/`v` lines
- `C`/`c` and `S`/`s` cubic Bézier curves
- `Q`/`q` and `T`/`t` quadratic Bézier curves
- `A`/`a` elliptical arcs
- `Z`/`z` close path

Clients must accept comma or whitespace separators, implicit command repetition, and the standard
rule that coordinate pairs after an initial move are line commands. A path may contain multiple
subpaths. Filling uses the
[SVG nonzero winding rule](https://www.w3.org/TR/SVG2/painting.html#FillRuleProperty); an open subpath
is implicitly closed for filling but remains open for stroking. A malformed `d` makes only that record
invalid under the normal [error-handling rules](#error-handling).

The field contains geometry only. SVG elements, styles, scripts, URLs, event attributes, and other
document features are not interpreted. Clients may normalize relative and shorthand commands or
flatten arcs to cubic Bézier segments while parsing.

### `text`

```json
{"kind": "text", "x": 160, "y": 24, "text": "Health", "size": 16,
 "color": "#ffffff", "align_x": "center", "align_y": "top", "font": "default"}
```

`text` is required and is one unwrapped line; CR and LF are invalid in version 1 text records. `x` and
`y` default to 0; `size` defaults to 16 scene units; `color` defaults to white. `align_x` is `left`,
`center`, or `right`; `align_y` is `top`, `center`, or `bottom`. Together they select which point of
the rendered text bounds is placed at `(x, y)`. `font` is `default` or `monospace`. Font metrics are
client-dependent, so critical geometry must not depend on exact text bounds.

### `group`

```json
{
  "kind": "group",
  "transform": {"translate": [10, 20], "rotate": 15, "scale": [1.5, 1.5]},
  "children": [
    {"kind": "rect", "width": 50, "height": 20, "fill": "#336699"}
  ]
}
```

`children` is required. `scale` may be one number or `[x, y]`. Transform components are listed as
translate, clockwise rotation in degrees, then scale about the group's local origin. In point order,
the transform is `parent = translate + rotate(scale(local))`. This definition takes precedence over
the call order used by a graphics API.

### Animations

Every drawing record may carry an `animate` object whose keys name properties of that record:

```json
{
  "kind": "circle",
  "id": "warning-pulse",
  "cx": 80,
  "cy": 80,
  "r": 8,
  "fill": "#ffcc33",
  "animate": {
    "r": {"to": 18, "duration": 700, "ease": "out", "repeat": "infinite"},
    "opacity": {"from": 1, "to": 0, "duration": 700, "repeat": "infinite"}
  }
}
```

Version 1 defines these tween targets:

| Record | Numeric targets | Color targets |
| --- | --- | --- |
| `rect` | `x`, `y`, `width`, `height`, `rx`, `opacity`, `stroke_width` | `fill`, `stroke` |
| `circle` | `cx`, `cy`, `r`, `opacity`, `stroke_width` | `fill`, `stroke` |
| `line` | `x1`, `y1`, `x2`, `y2`, `opacity`, `stroke_width` | `stroke` |
| `polyline` | `opacity`, `stroke_width` | `stroke` |
| `polygon`, `path` | `opacity`, `stroke_width` | `fill`, `stroke` |
| `text` | `x`, `y`, `size`, `opacity` | `color` |
| `group` | `translate_x`, `translate_y`, `rotate`, `scale`, `opacity` | — |
| `image` | `x`, `y`, `width`, `height`, `rotation`, `opacity` | — |

`translate_x` and `translate_y` address the two components of `transform.translate`; `rotate` and
`scale` address the corresponding group transform fields. Animated `scale` is uniform. A
`stroke_width` tween has no visible effect when the record has no stroke. Color tweens use solid
colors and interpolate red, green, blue, and alpha components independently.

Each tween has this form:

```json
{"from": 8, "to": 18, "duration": 700, "delay": 0,
 "ease": "out", "repeat": "infinite"}
```

| Property | Required | Description |
| --- | --- | --- |
| `to` | Yes | Finite number, or a color string for a color target. |
| `from` | No | Matching starting value. Defaults to the record's static value for that property. |
| `duration` | Yes | Non-negative duration of one repetition in milliseconds. |
| `delay` | No | Non-negative delay before the first repetition in milliseconds. Defaults to 0. |
| `ease` | No | `linear`, `in`, `out`, or `in-out`. Defaults to `linear`. |
| `repeat` | No | Integer from 1 through 4,294,967,295, or `infinite`. Defaults to 1. |

For `opacity`, both endpoints must be between 0 and 1. For group `scale`, an omitted `from` uses the
static numeric scale or the first component of a static `[x, y]` scale; playback then applies that
value uniformly. If a color target has no static color and omits `from`, it starts at `to`.

Each repetition runs from `from` to `to`; repetitions do not alternate direction. The initial delay
applies once. The easing names specify cubic easing: `in` is cubic acceleration, `out` is cubic
deceleration, and `in-out` applies the corresponding cubic curve to each half. A zero-duration tween
changes directly to `to` after its delay and does not require continuous redraws.

An animated record uses its `id` as its identity when that value is unique. Without a unique `id`, it
uses its shape kind and preorder position. Duplicate animation ids do not invalidate a `Set`. When a
valid `Set` replaces a canvas, an animation with the same identity and the same parsed `animate`
object keeps its elapsed clock. Changing the tween specification restarts it. Only the clock is
retained; all static properties come from the newest `Set`.

When `transient` is true, the record stops drawing after all its finite tweens finish.

Animation playback is optional, but static fallback is mandatory. A client that does not support
playback, has reduced-motion enabled, is in the background, or otherwise suppresses motion ignores
`animate` and `transient` and draws the static record. It must not skip an otherwise valid record, and
no capability advertisement is needed. A malformed `animate` object is handled the same way.

Playback is based on elapsed time rather than a required frame rate. Clients may limit redraw rate
or suspend playback when a canvas is not visible.

### Transitions

`transition` smooths changes to live properties, especially values supplied by GMCP bindings:

```json
{
  "kind": "rect",
  "width": {"$bind": "$data", "path": "/hp_width", "fallback": 0},
  "height": 18,
  "fill": {"$bind": "$data", "path": "/hp_color", "fallback": "#d14545"},
  "transition": {
    "width": {"duration": 250, "ease": "out"},
    "fill": {"duration": 150, "ease": "linear"}
  }
}
```

A transition may name any numeric or color target listed in the [animation table](#animations). Its
specification contains `duration`, and optionally `delay` and `ease`, with the same meanings as a
tween. It does not contain `from`, `to`, `repeat`, or `transient`; the property's previously displayed
and newly resolved values supply the endpoints.

The first valid value displays immediately. When that property later resolves to a different value,
the client interpolates from the value currently on screen to the new target. If another update
arrives before completion, the client retargets from the current interpolated value and uses the full
declared duration again. An identical target does not restart the transition. During `delay`, the
previously displayed value remains on screen.

Changes between a binding and its `fallback` also transition when both values are valid for the
property. If a required property becomes absent, the usual missing-property rules apply instead.

A property may be bound and transitioned. It must not appear in both `animate` and `transition`; if
it does, `animate` takes precedence and `transition` is ignored for that property. A malformed
transition also falls back to an immediate change.

Clients that do not support motion, have reduced-motion enabled, or suppress animation snap to the
latest resolved value. Transition support requires no separate capability advertisement.

### `image`

```json
{
  "kind": "image",
  "src": "https://game.example/portraits/ada.jpg",
  "alt": "Portrait of Ada wearing silver armor",
  "x": 16,
  "y": 16,
  "width": 96,
  "height": 128,
  "fit": "cover",
  "rotation": -3,
  "border_radius": [12, 12, 4, 4],
  "opacity": 0.9
}
```

| Property | Required | Description |
| --- | --- | --- |
| `src` | Yes | Absolute image URL. Version 1 clients must support HTTPS URLs. |
| `alt` | Yes | Plain-text alternative. Use an empty string only for a decorative image. |
| `x`, `y` | No | Top-left of the image bounds. Defaults to 0. |
| `width`, `height` | Yes | Image bounds in scene units. |
| `fit` | No | `contain`, `cover`, `fill`, `none`, or `scale-down`. Defaults to `contain`. |
| `rotation` | No | Clockwise rotation in degrees about the center. Defaults to 0. |
| `border_radius` | No | One radius or `[top-left, top-right, bottom-right, bottom-left]`. Defaults to 0. |
| `opacity` | No | Common record opacity, from 0 through 1. Defaults to 1. |

The `fit` values mean:

- `contain` preserves aspect ratio and fits the whole image inside the bounds.
- `cover` preserves aspect ratio and fills the bounds, cropping overflow.
- `fill` stretches the image to the exact bounds.
- `none` keeps the intrinsic image size and centers it in the bounds.
- `scale-down` behaves like `contain` only when the image is too large; it never scales up.

Each corner radius is clamped to half the smaller of `width` and `height`. Rotation does not change
the record's layout or link bounds. The client establishes the rounded, unrotated bounds as a clip,
rotates the fitted image clockwise about the center of those bounds, and draws it through that clip.

Version 1 clients must decode static PNG and JPEG images. They may support additional raster formats,
but servers use PNG or JPEG for portable canvases. SVG, animated images, `data:` URLs, and
`file:` URLs are not part of version 1. Intrinsic size means the decoded bitmap dimensions, with one
bitmap pixel equal to one scene unit for `none` fitting. Servers provide pixels in their intended
display orientation and do not rely on EXIF orientation or density metadata.

Images load asynchronously. A failed, refused, or still-loading image skips only that image record;
the rest of the accepted scene remains visible. Clients may show a native placeholder and should use
`alt` in accessible or text-only representations. Clients should reuse cached downloads and decoded
image resources when later snapshots use the same `src`. An asynchronous completion affects only a
current scene that still references that image; it must not resurrect an older snapshot.

Normal network policy still applies. A client may require user approval, block a host, reject
redirects that leave HTTPS, or impose download, decoded-pixel, cache, and time limits. Version 1
requires support for permitted HTTPS PNG and JPEG resources, not that every resource be fetched.
Downloaded image data must not be handed to an operating-system file or URL handler.

## GMCP bindings

A binding token replaces a scalar canvas property with a value retained from one GMCP message, or
from the canvas's private `$data` object:

```json
{"$bind": "Char.Vitals", "path": "/hp", "fallback": 0}
```

`$bind` is an object rather than string interpolation. Braces in ordinary text have no special
meaning.

For a textual property, `format` inserts the display value at one `{}` placeholder:

```json
{"$bind": "Char.Vitals", "path": "/hp", "format": "{} HP", "fallback": "--"}
```

| Property | Required | Description |
| --- | --- | --- |
| `$bind` | Yes | Exact GMCP message name to observe, or `$data` for canvas-local data. |
| `path` | No | JSON Pointer selecting within that message's payload. Defaults to the payload root. |
| `fallback` | No | Value used while the message or path is absent, `null`, or invalid for the destination property. |
| `format` | No | Text template containing exactly one `{}` placeholder. Valid only for textual properties. |

`path` uses [JSON Pointer (RFC 6901)](https://www.rfc-editor.org/rfc/rfc6901): it is either the empty
string for the payload root or begins with `/`; `~0` represents `~` and `~1` represents `/`. Array
elements may be selected with `0` or a positive decimal index without a leading zero. The special `-`
token does not resolve a value. For example:

```json
{"$bind": "Group.Members", "path": "/0/hp", "fallback": 0}
```

`$bind` names one GMCP message; `path` only walks that message's payload. This keeps GMCP name
segments separate from JSON keys and lets a client store binding data in a map keyed by message name.

Bindings are direct lookups, never expressions. They cannot perform arithmetic, comparisons,
conditionals, concatenation, command execution, or access client-local state. If a derived value such
as a health percentage is needed, the server publishes that value through GMCP and binds it directly.

### Retained GMCP data

The client maintains connection-scoped binding data for each exact GMCP message referenced by an
active canvas. For example:

```json
Char.Vitals {"hp": 823, "maxhp": 1000}
```

Both `$bind: "Char.Vitals"` with `path: "/hp"` and the same binding with `path: "/maxhp"` resolve
from that payload. A later payload for the same message updates the retained data as follows:

- When the old and new values at a location are both JSON objects, members present in the new object
  are updated recursively and members omitted from it are retained.
- An incoming array, string, number, boolean, or `null` replaces the old value at that location.
  Arrays are replaced as wholes; they are not merged by index.

After `Char.Vitals {"hp": 790}`, the retained `maxhp` above is still `1000`. A client must not lose a
bound value because a later object payload for the same message omits that member. To clear it, the
server sends the member as `null`; the binding then uses its `fallback` or becomes absent.

These merge rules apply only to `Client.Canvas` binding data. They do not change the GMCP package's
own semantics or the payloads exposed to scripts and other client features.

GMCP message-name matching is ASCII case-insensitive, while JSON Pointer object-key matching is
case-sensitive. Thus `$bind: "char.vitals"` with `path: "/hp"` resolves the example above, but
`path: "/HP"` does not.

A client may already have enough retained GMCP data to resolve a new binding immediately. A client
that retains only messages used by active canvases starts tracking them when it accepts the `Set`.
After introducing a binding, the server must send each value it relies on at least once.

### `Client.Canvas.Data` and `$data`

Servers may keep presentation-specific values alongside a canvas without inventing another GMCP
package:

```json
Client.Canvas.Data {
  "id": "vitals",
  "data": {
    "hp_width": 165,
    "hp_fraction": 0.823,
    "hp_color": "#d14545"
  }
}
```

`id` names an existing canvas and `data` is a JSON object. The newest valid `Data` message for that
canvas completely replaces its previous data; there are no patches, revisions, or acknowledgements.
A malformed message or one naming an unknown canvas is ignored, leaving any previous data unchanged.
The retained-GMCP object-merge rules do not apply to `$data`.

Bindings whose `$bind` value is `$data` resolve against the data belonging to their own canvas:

```json
{"$bind": "$data", "path": "/hp_fraction", "fallback": 0}
```

`$data` cannot read another canvas's private data. A replacement `Set` with the same canvas `id` retains
its data. `Client.Canvas.Remove` and the end of the connection clear it.

Data updates use the same latest-value repaint behavior as other bindings and may trigger
transitions. Established values should still come from their normal GMCP packages; for example,
`Char.Vitals` with `path: "/hp"`. `$data` is for derived values, presentation choices, and other
state private to one canvas.

### Resolution and repainting

A binding may replace any scalar leaf inside the `canvas` object or a drawing, stroke, image, text,
or link record. It cannot replace structural or identity fields such as `kind`, record `id`, `scene`,
`children`, `points`, `animate`, `transition`, or an entire `link` object. Binding objects inside
tween or transition definitions are not allowed.

Resolution preserves the source JSON scalar, with two conversions:

- A numeric destination accepts a JSON number or a string containing one finite JSON number within
  the version 1 numeric range, including vitals encoded as decimal strings.
- A textual destination displays strings directly, numbers and booleans in JSON spelling, and `null`
  as absent. `format`, when present, is applied after this conversion.

All other destinations use their normal validation rules. Arrays and objects do not resolve into
scalar properties. If resolution fails, `fallback` is tried under the same rules. Without a valid
fallback, an optional property uses its normal default; a record missing a required property is not
drawn until the binding resolves. This does not invalidate the `Set`. A malformed binding or JSON
Pointer is unresolved, but may still use a valid `fallback`.

When an inbound GMCP message matches a `$bind` value, the client resolves the newest value and
schedules a repaint. Multiple writes may be coalesced so long as the next rendered frame uses the
latest value. A binding update is not a new canvas snapshot, does not restart animations, may start
or retarget transitions, and requires no acknowledgement or `Client.Canvas.Set` from the server.
An omitted object member leaves its resolved value unchanged and does not by itself start or retarget
a transition.

A property may be both bound and named in its record's `transition` object. A property must not be
both bound and named in `animate`; if both are present, the binding supplies the property and that
tween is ignored. Replacing or removing a canvas also removes its old bindings; binding state never
crosses connections.

## Links and actions

A link makes a drawing record activatable. It borrows OSC 8's URI-plus-identity shape. Its MUD
command schemes follow Muddown, LociTerm, and established client behavior:

- [OSC 8 hyperlinks](https://iterm2.com/feature-reporting/Hyperlinks_in_Terminal_Emulators.html)
- [Muddown command links](https://www.muddown.com/)
- [LociTerm OSC 8 `send:` and `prompt:` links](https://github.com/RahjIII/lociterm/blob/dev/docs/osc8hyperlinks.txt)
- [MXP `SEND` links](https://wiki.mudlet.org/images/c/ca/MUD_eXtension_Protocol.pdf)

```json
{
  "href": "send:drink health",
  "id": "drink-health",
  "label": "Drink a health potion",
  "hint": "Consumes one potion"
}
```

A GMCP action uses the same message-name and JSON spelling as a normal GMCP payload:

```json
{
  "href": "gmcp:Game.Ui.Action {\"id\":\"drink-health\"}",
  "label": "Drink a health potion"
}
```

| Property | Required | Description |
| --- | --- | --- |
| `href` | Yes | Action target. Maximum 2048 UTF-8 bytes. |
| `id` | No | Logical identity shared by related, non-adjacent linked records. Maximum 128 UTF-8 bytes. |
| `label` | Yes | Accessible action label. |
| `hint` | No | Additional text the client may display on hover, focus, or long press. |

Version 1 defines these schemes:

- `send:` sends everything after the prefix to the MUD as a command when the user activates it.
- `prompt:` places everything after the prefix in the client's command input without sending it.
- `gmcp:` sends a GMCP message to the MUD when the user activates it.
- `http:` and `https:` request opening the URL under the client's normal external-link policy.

Scheme matching is ASCII case-insensitive, but servers use the lowercase spellings above. The
`send:`, `prompt:`, and `gmcp:` payloads are literal UTF-8 and are not percent-decoded. Unknown schemes
are inert. Clients must not treat them as operating-system URLs or dispatch them to an installed
application.

A `send:` payload is one command line and must not contain CR or LF. The client transmits it exactly
once, followed by the connection's normal command terminator, without local alias expansion. A
`prompt:` payload also contains no CR or LF; normal client behavior applies if the user later edits
and submits it.

A `gmcp:` payload contains a GMCP message name and, optionally, one ASCII space followed by a valid
JSON value. For example, `gmcp:Game.Ui.Close` sends a message without data, while
`gmcp:Game.Ui.Select {"id":7}` sends the object as its data. The client validates the name and JSON,
constructs the GMCP subnegotiation itself, and sends it only while GMCP is enabled on the current
connection. A malformed payload or a `gmcp:` link activated without GMCP support is inert. It is not
submitted as a command, passed through aliases, or interpreted as an inbound message by the client.

Links activate only through deliberate user input. A client should expose them to pointer, keyboard,
touch, and assistive-technology users. It may require a modifier or confirmation and should expose
the target or `hint` before activation.

A shape's base link region is its visibly filled and stroked geometry. Text uses its client-measured
text bounds and an image uses its unrotated bounds. A link on a group applies to the union of its
descendants' base link regions; a descendant's own link overrides the inherited link. During
playback, a record's region follows its displayed animated geometry; when playback is suppressed it
follows the static geometry. When linked records overlap, the topmost rendered linked record wins.
A client may enlarge a small region to provide a usable pointer or touch target and may present
records sharing a link `id` as one logical action. A record with no visible fill, stroke, text, or
image must not become an invisible link-capture surface.

There is no `Client.Canvas.Event` in version 1. Normal MUD commands use the existing command stream;
external links use the client's existing link handling.

## Size and resource limits

Version 1 has three fixed limits:

- The UTF-8 JSON body of any `Client.Canvas` message must not exceed 256 KiB.
- A scene may nest groups at most 16 levels deep.
- A connection may have at most 32 active canvas identifiers.

Servers must remain within these limits. Clients must accept valid version 1 messages within them and
may accept larger messages as an extension. Client parser, allocation, and renderer limits must be at
least this large.

Once 32 canvas identifiers are active, a `Set` that replaces one of them is processed normally but a
`Set` with a new identifier may be ignored. `Remove` or closing a target frame frees that identifier.

Remote images are not included in the JSON byte limit.

## Error handling

Unknown top-level properties and unknown properties on known records are ignored for forward
compatibility. An unknown `kind`, malformed color, unavailable image, or otherwise invalid individual
record is skipped without changing the order of valid records.

An invalid top-level object or a scene exceeding a fixed conformance limit rejects the whole `Set`.
The previous valid canvas remains visible. Diagnostics are local to the client; version 1 defines no
acknowledgement or error message.
