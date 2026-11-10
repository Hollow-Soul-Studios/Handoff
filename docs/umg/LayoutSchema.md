# Handoff package format (`layout.json` schema v1)

A Handoff package is the only contract between an exporter (the Figma plugin, the
optional REST fetcher, future Photoshop/Penpot exporters) and the Unreal importer.
It is tool-neutral: nothing in it requires Figma.

## 1. Container

A package is either a `.handoff.zip` archive (stored or deflate entries) or a folder
with the same layout:

```
layout.json
assets/<assetId>@1x.png
assets/<assetId>@2x.png
assets/<assetId>.svg          (optional, informational)
```

Paths inside `layout.json` are relative to the package root and always use `/`.

## 2. Versioning

`schemaVersion` is an integer. The importer accepts the current version (`1`) and the
previous one, and warns when a package was written by a newer exporter
(`schemaVersion` greater than the importer's current version: import is attempted and
unknown fields are ignored).

## 3. Determinism rules for exporters

* Arrays keep document order (Figma layer order is bottom-to-top; exporters emit
  children **back-to-front**, i.e. index 0 is drawn first).
* Object keys may appear in any order; the importer never depends on key order.
* Asset ids are derived from content (hash of the PNG bytes or the Figma image hash),
  so identical images share one asset.
* `exportedAt` is informational and never influences generated assets.

## 4. Top level

```jsonc
{
  "schemaVersion": 1,
  "generator": { "name": "handoff-figma", "version": "1.0.0" },
  "source": {
    "tool": "figma",               // "figma" | "figma-rest" | "photoshop" | "penpot" | ...
    "fileKey": "AbC123",           // stable document id ("" if unknown)
    "fileName": "Game UI",
    "pageName": "Menus",
    "exportedAt": "2026-11-20T10:00:00Z",
    "colorProfile": "SRGB"         // optional: "SRGB" | "DISPLAY_P3"; informational only
  },
  "frames":        [ Node ],        // top-level frames -> one Widget Blueprint each
  "components":    [ Component ],   // main components referenced by INSTANCE nodes
  "componentSets": [ ComponentSet ],
  "textStyles":    [ TextStyle ],
  "paintStyles":   [ PaintStyle ],
  "variables":     { "collections": [ VariableCollection ], "variables": [ Variable ] },
  "fonts":         [ { "family": "Inter", "style": "Bold", "weight": 700, "italic": false } ],
  "assets":        [ Asset ]
}
```

All arrays are optional (missing = empty). `frames` must contain at least one node.

`source.colorProfile` (optional) records the colour profile of the source document. The importer does not convert
colours: values are always read as sRGB, so `DISPLAY_P3` documents may look slightly different in Unreal.

## 5. Node

```jsonc
{
  "id": "12:34",                  // tool node id; instance children look like "I12:40;3:5"
  "name": "Play Button",          // layer name
  "type": "FRAME",                // FRAME | GROUP | SECTION | COMPONENT | COMPONENT_SET | INSTANCE |
                                  // RECTANGLE | ELLIPSE | LINE | VECTOR | STAR | POLYGON |
                                  // BOOLEAN_OPERATION | TEXT
  "visible": true,
  "opacity": 1.0,                 // 0..1 layer opacity
  "x": 0, "y": 0,                 // px, relative to the parent node's top-left, unrotated box
  "width": 100, "height": 40,     // px
  "rotation": 0,                  // degrees, clockwise positive, about the box centre
  "constraints": { "horizontal": "MIN", "vertical": "MIN" },
                                  // MIN | MAX | CENTER | STRETCH | SCALE
                                  // (GROUP: exporter emits the dominant child constraint)
  "clipsContent": false,
  "fills":   [ Paint ],
  "strokes": [ Paint ],
  "strokeWeight": 0,
  "strokeAlign": "INSIDE",        // INSIDE | OUTSIDE | CENTER
  "cornerRadius": [0, 0, 0, 0],   // top-left, top-right, bottom-right, bottom-left (px)
  "effects": [ Effect ],
  "fillStyleId": "S:abc,",        // optional style references
  "strokeStyleId": "",
  "textStyleId": "",
  "boundVariables": { "fills/0": "VariableID:1:2" },   // optional, informational

  "layout": AutoLayout,           // present only when the node uses auto layout
  "layoutChild": LayoutChild,     // present only when the parent uses auto layout
  "text": TextData,               // present only for TEXT nodes

  "asset":     { "id": "a_1f2e...", "reason": "vector" },
      // The whole node (including children) was rasterised. The importer emits a
      // single Image and ignores `children`. reason: vector | boolean | blur | mask |
      // gradient | image | effects | tagged
  "fillAsset": { "id": "a_9c0d..." },
      // Only this node's own fills/strokes were rasterised (children removed before
      // export). Used for containers with image or gradient backgrounds.

  "componentId": "3:1",           // INSTANCE: id of the main component (see `components`)
  "componentSetId": "3:0",        // COMPONENT inside a set
  "variantProperties": { "State": "Hover" },

  "handoff": HandoffTags,         // optional designer tags (pluginData or layer-name tags)
  "children": [ Node ]            // back-to-front
}
```

### Paint

```jsonc
{
  "type": "SOLID",                // SOLID | GRADIENT_LINEAR | GRADIENT_RADIAL | GRADIENT_ANGULAR |
                                  // GRADIENT_DIAMOND | IMAGE | VIDEO
  "visible": true,
  "opacity": 1.0,
  "color": { "r": 1, "g": 1, "b": 1, "a": 1 },      // sRGB, 0..1 (SOLID)
  "gradientStops": [ { "position": 0, "color": { "r":0,"g":0,"b":0,"a":1 } } ],
  "assetId": "img_<imageHash>",   // IMAGE
  "scaleMode": "FILL"             // FILL | FIT | TILE | STRETCH | CROP
}
```

Exporter contract: a node whose visible fills are anything other than **zero or one
SOLID paint** must carry `fillAsset` (containers) or `asset` (leaves). The importer
uses the first visible SOLID paint otherwise, and falls back to the first gradient
stop colour with a warning.

### Effect

```jsonc
{ "type": "DROP_SHADOW", "visible": true, "color": {"r":0,"g":0,"b":0,"a":0.25},
  "offset": {"x": 0, "y": 4}, "radius": 8, "spread": 0 }
// type: DROP_SHADOW | INNER_SHADOW | LAYER_BLUR | BACKGROUND_BLUR
```

Nodes with LAYER_BLUR / BACKGROUND_BLUR / INNER_SHADOW are rasterised by the exporter
(`asset.reason = "blur" | "effects"`). DROP_SHADOW on TEXT becomes a text shadow; on
other nodes it is reported as unsupported unless rasterised.

### AutoLayout

```jsonc
{
  "mode": "HORIZONTAL",           // HORIZONTAL | VERTICAL | WRAP (horizontal + wrap) | GRID
  "padding": { "left": 0, "top": 0, "right": 0, "bottom": 0 },
  "itemSpacing": 0,               // gap along the primary axis (px)
  "counterAxisSpacing": 0,        // gap between wrapped rows (WRAP)
  "primaryAxisAlign": "MIN",      // MIN | CENTER | MAX | SPACE_BETWEEN
  "counterAxisAlign": "MIN",      // MIN | CENTER | MAX | BASELINE
  "primaryAxisSizing": "FIXED",   // FIXED | AUTO (hug)
  "counterAxisSizing": "FIXED"    // FIXED | AUTO (hug)
}
```

### LayoutChild

```jsonc
{
  "positioning": "AUTO",          // AUTO | ABSOLUTE (absolute children use `constraints`)
  "grow": 0,                      // 1 = fill along the parent's primary axis
  "align": "INHERIT",             // INHERIT | STRETCH (fill along the counter axis)
  "horizontalSizing": "FIXED",    // FIXED | HUG | FILL
  "verticalSizing": "FIXED"       // FIXED | HUG | FILL
}
```

### TextData

```jsonc
{
  "characters": "Play",
  "fontFamily": "Inter",
  "fontStyle": "Bold",            // Figma style name; also the composite-font typeface name
  "fontWeight": 700,
  "italic": false,
  "fontSize": 32,                 // px
  "lineHeight": { "unit": "AUTO", "value": 0 },     // AUTO | PIXELS | PERCENT (of font size)
  "letterSpacing": { "unit": "PERCENT", "value": 0 }, // PIXELS | PERCENT (of font size)
  "textCase": "ORIGINAL",         // ORIGINAL | UPPER | LOWER | TITLE
  "textDecoration": "NONE",       // NONE | UNDERLINE | STRIKETHROUGH
  "fills": [ Paint ],
  "alignHorizontal": "LEFT",      // LEFT | CENTER | RIGHT | JUSTIFIED
  "alignVertical": "TOP",         // TOP | CENTER | BOTTOM
  "autoResize": "NONE",           // NONE | WIDTH_AND_HEIGHT | HEIGHT | TRUNCATE
  "maxLines": 0,                  // 0 = unlimited
  "mixed": false                  // true when the layer mixes styles; first run is exported
}
```

### HandoffTags

Designers tag layers either with plugin data written by the exporter UI or with
bracketed tags in the layer name, e.g. `Play [button] [focus]`. The exporter strips
tags from `name` and emits them here.

```jsonc
{
  "widget": "Border",             // force a UMG class: Border | Image | SizeBox | Button | Spacer | Overlay
  "role": "button",               // button | ignore | rasterize | safezone | screen | component
                                  // role "safezone" is equivalent to safeZone: true (both are accepted)
  "defaultFocus": true,           // [focus]  default gamepad focus target of the screen
  "nineSlice": { "left": 16, "top": 16, "right": 16, "bottom": 16 },  // px  [9slice=16] or [9slice=l,t,r,b]
  "inputAction": "IA_Confirm",    // [action=IA_Confirm] CommonUI triggering input action
  "safeZone": true,               // [safezone] wrap the screen in a SafeZone
  "inline": false                 // [inline] expand an instance instead of using the component widget
}
```

## 6. Component and component set

```jsonc
// Component
{ "id": "3:1", "key": "c0ffee...", "name": "Button/Primary", "description": "",
  "componentSetId": "3:0", "variantProperties": { "State": "Default" },
  "node": Node }                  // node.type == "COMPONENT", coordinates relative to itself (x = y = 0)

// ComponentSet
{ "id": "3:0", "key": "...", "name": "Button", "role": "button",
  "variantProperties": { "State": ["Default", "Hover", "Pressed", "Disabled"] },
  "componentIds": ["3:1", "3:2", "3:3", "3:4"] }
```

A component set is treated as a **button** when `role == "button"` or when it has a
`State` variant property. State values map case-insensitively:
`default|normal|rest|enabled` -> Normal, `hover|hovered` -> Hovered,
`pressed|active|down` -> Pressed, `disabled|inactive` -> Disabled,
`selected` -> Selected, `selected hover` -> SelectedHovered, `selected pressed` -> SelectedPressed.
Other variant properties produce one component widget per variant.

Instance nodes carry their resolved children (ids `I<instanceId>;<componentChildId>`),
so an importer can expand an instance inline. Text overrides are detected by comparing
an instance child's `text.characters` with the component node whose id equals the last
`;`-separated segment of the child id.

## 7. Styles and variables

```jsonc
// TextStyle
{ "id": "S:ab12,", "key": "...", "name": "Heading/H1",
  "fontFamily": "Inter", "fontStyle": "Bold", "fontWeight": 700, "italic": false, "fontSize": 48,
  "lineHeight": { "unit": "PIXELS", "value": 56 }, "letterSpacing": { "unit": "PERCENT", "value": -2 },
  "textCase": "ORIGINAL", "textDecoration": "NONE",
  "fill": { "r": 1, "g": 1, "b": 1, "a": 1 } }          // optional default colour

// PaintStyle
{ "id": "S:cd34,", "key": "...", "name": "Brand/Primary", "paints": [ Paint ] }

// VariableCollection
{ "id": "VariableCollectionId:1:1", "name": "Tokens", "defaultModeId": "1:0",
  "modes": [ { "id": "1:0", "name": "Dark" } ] }

// Variable
{ "id": "VariableID:1:2", "name": "color/primary", "collectionId": "VariableCollectionId:1:1",
  "type": "COLOR",                // COLOR | FLOAT | STRING | BOOLEAN
  "valuesByMode": { "1:0": { "r": 1, "g": 0.4, "b": 0, "a": 1 } } }
  // an aliased value is { "alias": "VariableID:1:9" }
```

## 8. Asset

```jsonc
{ "id": "a_1f2e3d4c5b6a7980", "name": "Logo", "kind": "vector",   // image | vector | raster
  "width": 128, "height": 64,                                      // 1x size in px
  "files": { "png1x": "assets/a_1f2e3d4c5b6a7980@1x.png",
             "png2x": "assets/a_1f2e3d4c5b6a7980@2x.png",
             "svg":   "assets/a_1f2e3d4c5b6a7980.svg" } }
```

At least one of `png1x` / `png2x` is required.

## 9. Import receipt (write-back)

After an import the Unreal importer writes `<package>.receipt.json` next to the package
(or in the chosen folder). The exporter's "Load import receipt" command reads it and
stores the result in each node's `pluginData` (`handoff.status`, `handoff.ueAsset`,
`handoff.ueWidget`).

```jsonc
{
  "receiptVersion": 1,
  "fileKey": "AbC123",
  "importedAt": "2026-11-20T10:05:00Z",
  "engineVersion": "5.8.4",
  "nodes": [
    { "id": "12:34", "status": "imported", "asset": "/Game/UI/Handoff/WBP_MainMenu",
      "widget": "fig_8d2c11aa" }
    // status: imported | updated | conflict | kept | removed | skipped
  ]
}
```
