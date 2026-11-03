# Feature mapping: design element -> UMG output

Rules live in `Source/HandoffImporter/Private/HandoffWidgetMapper.cpp`. "CommonUI mode" is the import option
(needs the Common UI plugin); a blank cell means the same as plain UMG.

## Screens and layout

| Design element | Plain UMG | CommonUI mode |
|---|---|---|
| Top-level frame | Widget Blueprint `WBP_<Frame>`, parent `UserWidget`, root Canvas Panel filling the viewport | Parent `CommonActivatableWidget` |
| Frame tagged `[safezone]` (or option *Safe zone*; schema `safeZone: true` / `role: "safezone"`) | Root Canvas Panel holds the frame background and leading full-bleed layers (SCALE/STRETCH on both axes, covering the frame), then a Safe Zone with the content | |
| Frame without auto layout | Canvas Panel; background fill becomes a stretched Image child `#bg` | |
| Auto layout HORIZONTAL / VERTICAL | Horizontal Box / Vertical Box | |
| Auto layout WRAP, GRID | Wrap Box (inner slot padding, wrap size). Grid is approximated by a Wrap Box | |
| Padding or background on an auto-layout frame | Border `#bg` | |
| Item spacing | Leading padding on each slot after the first | |
| Fill / grow child | Slot size Fill | |
| Counter-axis align / stretch | Slot horizontal/vertical alignment | |
| Primary axis CENTER / MAX / SPACE_BETWEEN on a fixed-size box | Fill Spacers (`gapN`) | |
| FIXED-size child in a box | Size Box `#size` override | |
| ABSOLUTE child of an auto-layout frame | Overlay (`#stack`) containing a Canvas Panel (`#abs`) | |
| Constraints (non-auto-layout parent) | Canvas Panel slot anchors: MIN 0, MAX 1, CENTER 0.5, STRETCH 0..1 with insets, SCALE = fractions | |
| Opacity / rotation | Render Opacity / Render Transform angle | |
| Hidden layer | Visibility Collapsed | |
| Clip content | Clip To Bounds | |
| Hit testing | Containers Self Hit Test Invisible, leaves Hit Test Invisible, buttons and instances Visible | |

## Text

| Design element | Plain UMG | CommonUI mode |
|---|---|---|
| Text layer | Text Block | Common Text Block with a Style when the node matches its text style exactly (colour included); otherwise inline typography plus an info message |
| Font size | px x 0.75 = Slate points | |
| Letter spacing | per-mille | |
| Line height | Line Height Percentage using the font's hhea metrics (needs the font mapped) | |
| Alignment | Justification; vertical alignment through Border `#valign` | |
| Auto-resize | Wrap text unless WIDTH_AND_HEIGHT (then auto-size slot); TRUNCATE -> ellipsis overflow | |
| UPPER / LOWER | Text Transform Policy | |
| Drop shadow on text | Shadow offset and colour (blur ignored) | |
| Text content | Stable `NSLOCTEXT("Handoff", key, text)` keys | |
| Font | Mapped font asset, see [FontsAndStyles.md](FontsAndStyles.md); unmapped -> engine default font plus a warning | |

Not imported: mixed styles (first run only), justified (left-aligned), title case, underline, strikethrough,
shadow blur.

## Shapes and images

| Design element | Output |
|---|---|
| Rasterised node (`asset`: vectors, booleans, blur, masks, gradients, tagged `[rasterize]`) | Image with Texture brush; children ignored |
| Container with image/gradient background (`fillAsset`) | Image background with Texture brush (`#bg`) |
| Tag `[9slice=16]` / `[9slice=l,t,r,b]` | Brush Draw As Box with those margins |
| Solid rectangle | Image; Rounded Box with corner radii and outline for strokes |
| Circle (ellipse with equal sides) | Image, rounded box with half-height radius |
| Non-circular ellipse without an asset | Drawn as a pill |
| Invisible rectangle | Size Box |
| `[widget=Border]` | Border |
| LINE | Thin Image |
| Colours | sRGB converted to linear (Display P3 is not converted) |

Textures are named `T_<Name>_<hash6>`, TEXTGROUP_UI, no mips, UserInterface2D, never streamed. The 1x or 2x PNG
is used according to the *Textures* option; the brush keeps the 1x size.

## Components

| Design element | Plain UMG | CommonUI mode |
|---|---|---|
| Component / set (variant) | Child Widget Blueprint `Components/WBP_<Name>` | |
| Component text layer | Instance-editable, expose-on-spawn `FText` variable `Text_<Layer>`, set by a Handoff-owned function `HandoffApplyText` called from Event PreConstruct | |
| Instance | Instance of the component widget; overridden texts set the variables; Disabled variants set `bIsEnabled=false` | |
| Instance tagged `[inline]`, or option *Components as widgets* off | Children expanded in place | |
| Button set (`role: button`, `[button]`, or a `State` variant) | One widget per set: `UButton` root with per-state `FButtonStyle` brushes and padding | `UCommonButtonBase` subclass with a generated button style `Styles/BS_<Set>` (per-state brushes, per-state text styles `TS_<Set>_<State>`, ButtonPadding, MinWidth/MinHeight) and a Handoff-owned function `HandoffApplyTextStyle` bound to OnCurrentTextStyleChanged |
| `[action=IA_X]` | Warning "needs CommonUI mode"; ignored | UE 5.8: Triggering Enhanced Input Action, found by name among `UInputAction` assets in *Input action folder* (enable Enhanced Input support in Common Input settings). UE 5.7: row of the legacy input action data table |
| `[focus]` | | Sets the screen's `DesiredFocusWidget` (used by `CommonActivatableWidget`) |

State names map case-insensitively: default/normal/rest/enabled, hover/hovered, pressed/active/down,
disabled/inactive, selected, selected hover, selected pressed.

## Tags

| Tag | Effect |
|---|---|
| `[button]` | Treat as a button |
| `[focus]` | Default focus target |
| `[9slice=16]`, `[9slice=l,t,r,b]` | 9-slice margins in px |
| `[action=IA_X]` | Input action for the button |
| `[safezone]` | Wrap the screen in a Safe Zone |
| `[inline]` | Expand an instance instead of using the component widget |
| `[ignore]` | Skip the layer |
| `[rasterize]` | Export as an image |
| `[widget=Border\|Image\|SizeBox\|Spacer\|Button]` | Force a widget class |

## Generated structure names

Primary widgets (one per design node) are Blueprint variables named `fig_<8 hex of FNV-1a of node id>`.
Helper wrappers carry a role suffix and are not variables: `_size` (Size Box), `_bg` (Border or Image),
`_stack` (Overlay), `_abs` (Canvas Panel), `_valign` (Border), `_root` / `_safe` (safe-zone screens),
`_button`, `_content`, `_gapN` (Spacers). Display labels are the layer names.
