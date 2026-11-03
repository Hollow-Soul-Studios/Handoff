# Code generation: design element -> Slate C++

Handoff for Slate turns a design package (`Docs/LayoutSchema.md`) into ordinary Slate code: `SCompoundWidget`
classes built with `SNew`/`SAssignNew`, a style class for fonts and text styles, and image/font files.
Rules live in `Source/SlateHandoffEditor/Private/SlateHandoffMapper.cpp` (design -> widget tree) and
`SlateHandoffCodeGen.cpp` (widget tree -> C++). The preview window interprets the same widget trees with live
widgets (`SlateHandoffPreview.cpp`), so what you preview is what the code builds.

## Files

`D` is the destination source folder (inside a module, e.g. `Source/MyGame/UI/Handoff`), `R` the resources
folder (default `Content/Slate/Handoff` of the project, or `Resources/Handoff` of the plugin that owns the
module).

| File | Owner | Written |
|---|---|---|
| `D/Generated/<Style>.h/.cpp` (e.g. `HandoffUIStyle.h` for `FHandoffUIStyle`) | Handoff | every import, merged with your edits |
| `D/Generated/S<Screen>Base.h/.cpp` | Handoff | every import, merged with your edits |
| `D/Generated/Components/S<Component>Base.h/.cpp` | Handoff | every import, merged with your edits |
| `D/S<Screen>.h/.cpp` | **you** | once, when missing; never touched again |
| `D/Components/S<Component>.h/.cpp` | **you** | once, when missing; never touched again |
| `R/Images/<Name>_<hash6>.png` | Handoff | when the image content changes |
| `R/Fonts/<Family>-<Style>.ttf` (or `.otf`), `R/Fonts/<Family>-OFL.txt` | Handoff | when the mapped font file changes |

All generated text uses LF line endings, tabs, ends with a newline and is ASCII only (non-ASCII text is
written as `\uXXXX` escapes). Includes between generated and user files are paths relative to the including
file.

Your module must depend on `Slate` and `SlateCore` (plus `Projects` when the module belongs to a plugin);
**Check Setup** reports a missing dependency with the line to add.

## Names

| Thing | Rule | Example |
|---|---|---|
| Screen class | `S` + class prefix + PascalCase(frame name); `2`, `3`... on a clash with an existing class or an engine/project header | `Main Menu` -> `SMainMenu` |
| Component class | PascalCase(set name + variant name) for plain components, PascalCase(set name) for button sets | `SPrimaryButton` |
| Generated base | `<Class>Base` | `SMainMenuBase` |
| Widget members | `fig_<8 hex FNV-1a of node id>`, frozen in the manifest | `fig_1a2b3c4d` |
| Wrapper widgets | same with a role suffix, created with `SNew` (not members) | `fig_1a2b3c4d_bg` |
| Brush members | `Brush_<widget name>` | `Brush_fig_1a2b3c4d_bg` |
| Button style members | `ButtonStyle_<widget name>` | `ButtonStyle_fig_9f9f9f9f_button` |
| Component text arguments | `Text_` + Identifier(layer name), `_2`... on duplicates | `Text_Label` |
| Text style entries | `Text.` + Identifier(style name) | `Text.Heading_H1` |
| Localisation | `NSLOCTEXT("Handoff", "<Class>.<8 hex of the member name>", "...")`; component arguments append `.Text_<Layer>`. Keys follow the frozen member name, so a re-linked layer keeps its key | `SMainMenu.6724ca12` |

Class names and member names are created once and then frozen in the manifest: renaming a frame or layer in
Figma never renames a class or member that your code uses.

Roles: `size` (SBox), `bg` (SBorder or SImage), `stack` (SOverlay), `abs` (SConstraintCanvas), `valign` (SBox),
`root` / `safe` (safe-zone screens), `button` (SButton root of a button class, a member), `content`,
`gapN` (SSpacer).

## Screens and layout

| Design element | Slate |
|---|---|
| Top-level frame | Screen class; the frame becomes an `SConstraintCanvas` in `ChildSlot`, stretched to the widget |
| Frame tagged `[safezone]` (or option *Safe zone*; schema `safeZone: true` / `role: "safezone"`) | Root `SConstraintCanvas` (`root`) with the frame background and leading full-bleed layers, then an `SSafeZone` (`safe`) with the content |
| Frame without auto layout | `SConstraintCanvas`; its background fill is a stretched `SImage` child (`bg`) |
| Auto layout HORIZONTAL / VERTICAL | `SHorizontalBox` / `SVerticalBox` |
| Auto layout WRAP, GRID | `SWrapBox` (`InnerSlotPadding`, `PreferredSize` for fixed width, `UseAllottedSize` for hug). Grid is approximated |
| Padding or background on an auto-layout frame | `SBorder` (`bg`) |
| Item spacing | Leading slot padding on each slot after the first |
| Fill / grow child | Slot `FillWidth(1.f)` / `FillHeight(1.f)` |
| Counter-axis align / stretch | Slot `HAlign` / `VAlign` |
| Primary axis CENTER / MAX / SPACE_BETWEEN on a fixed-size box | Fill `SSpacer`s (`gapN`) |
| FIXED-size child in a box | `SBox` (`size`) with `WidthOverride` / `HeightOverride` |
| ABSOLUTE child of an auto-layout frame | `SOverlay` (`stack`) holding the box and an `SConstraintCanvas` (`abs`) |
| Constraints (non-auto-layout parent) | `SConstraintCanvas` slot: `Anchors` MIN 0, MAX 1, CENTER 0.5, STRETCH 0..1 with insets, SCALE = fractions; `Offset`; `Alignment` (always emitted, see below); `AutoSize` for auto-width text |
| Opacity / rotation | `RenderOpacity` / `RenderTransform` (+ `RenderTransformPivot` 0.5, 0.5) |
| Hidden layer | `EVisibility::Collapsed` |
| Clip content | `EWidgetClipping::ClipToBounds` |
| Hit testing | Containers `SelfHitTestInvisible`, leaves `HitTestInvisible`, buttons and instances `Visible` |

## Text

| Design element | Slate |
|---|---|
| Text layer | `STextBlock` |
| Matches a document text style exactly (typography and colour) | `.TextStyle(&Style::Get().GetWidgetStyle<FTextBlockStyle>("Text.<Name>"))` |
| Otherwise | inline `Font`, `ColorAndOpacity`, `ShadowOffset`, `ShadowColorAndOpacity`, `TransformPolicy`, plus an info message naming the overridden style |
| Font size | px x 0.75 = Slate points |
| Letter spacing | per-mille (`Style::Font(..., LetterSpacing)`) |
| Line height | `LineHeightPercentage` from the font's hhea metrics (needs the font mapped; 1.2 assumed otherwise) |
| Alignment | `Justification`; vertical alignment through an `SBox` (`valign`) |
| Auto-resize | `AutoWrapText` unless WIDTH_AND_HEIGHT (then `AutoSize` canvas slot); TRUNCATE -> `ETextOverflowPolicy::Ellipsis` |
| UPPER / LOWER | `TransformPolicy` |
| Drop shadow | `ShadowOffset` and `ShadowColorAndOpacity` (blur ignored) |
| Font | `Style::Font("Family", "Style", Size)`; unmapped families fall back to the engine default font at runtime and the import warns |

Not imported: mixed styles (first run only), justified (left-aligned), title case, underline, strikethrough,
shadow blur.

## Shapes and images

| Design element | Slate |
|---|---|
| Rasterised node (`asset`: vectors, booleans, blur, masks, gradients, `[rasterize]`) | `SImage` with an image brush from the style class (`FHandoffUIStyle::Get().GetBrush("Image.<stem>.<W>x<H>")`) and `DesiredSizeOverride` = the design size; children ignored |
| Container with an image/gradient background (`fillAsset`) | `SImage` background (`bg`) with an image brush from the style class |
| `[9slice=16]` / `[9slice=l,t,r,b]` | Style entry `Box.<stem>.<W>x<H>.<4 hex of the margins>`: an `FSlateBoxBrush` with the margins as fractions of the image |
| Solid rectangle without radius or stroke | `SImage` with `FSlateColorBrush` |
| Rounded rectangle, stroke | `FSlateRoundedBoxBrush(fill, radii, outline colour, outline width)` |
| Circle (ellipse with equal sides) | `FSlateRoundedBoxBrush(fill, outline colour, outline width)` (half-height radius) |
| Non-circular ellipse without an asset | drawn as a pill, with a warning |
| Invisible rectangle | `SBox` |
| `[widget=Border]` | `SBorder` |
| LINE | thin `SImage` in the stroke colour |
| Colours | sRGB converted to linear (Display P3 is not converted) |

Images are copied to `R/Images/T_<Name>_<hash6>.png`; the 1x or 2x PNG according to the *Textures* option,
the brush keeps the 1x size. Slate loads image files only for brushes of a registered style set, so an image brush is
never created inline in a widget: it is an entry of the style class (`Style->Set("Image.T_Logo_f079c3.128x128",
new FSlateImageBrush(...))`) and widgets reference it by name. A tinted brush (opacity below 1) gets the suffix
`.Tint<percent>`. Rounded, colour and solid brushes need no file and stay inline `Brush_<widget>` members.
Every `SImage` also gets `DesiredSizeOverride` (the design size; for a line the stroke weight as height),
because colour and rounded-box brushes have no size of their own and an `SImage` in an auto-sized slot would
collapse without it.

## Canvas slot alignment

`SConstraintCanvas::FSlot` defaults `Alignment` to (0.5, 0.5), not (0, 0). The generator therefore always emits
`.Alignment(FVector2D(x, y))` on canvas slots: (0, 0) normally, and the text alignment fraction on the growing
axis for auto-sized text (its offset is shifted so the text grows from its aligned edge).

## Components

| Design element | Slate |
|---|---|
| Component / variant | Component class `S<Name>` (+ `S<Name>Base`) |
| Component text layer | `SLATE_ARGUMENT(FText, Text_<Layer>)` with the design text as default; the inner `STextBlock` uses `InArgs._Text_<Layer>` |
| Instance | `SAssignNew(fig_..., S<Name>)` with the overridden texts as arguments; Disabled variants add `.IsEnabled(false)` |
| Instance tagged `[inline]`, or option *Components as widgets* off | children expanded in place |
| Button set (`role: button`, `[button]`, or a `State` variant) | One class per set: `SButton` root (`button`) with an `FButtonStyle` member (Normal/Hovered/Pressed/Disabled brushes from the variants, NormalPadding/PressedPadding from the padding), `.ContentPadding(FMargin(0.f))`, `SLATE_EVENT(FOnClicked, OnClicked)` and `SetOnClicked()` |
| Frame tagged `[button]` (not a component) | `SButton` with the frame background in its style |
| `[action=IA_X]` | Not generated (bind input in your code); info message |
| `[focus]` | `GetDefaultFocusWidget()` of the screen returns that member |

State names map case-insensitively: default/normal/rest/enabled, hover/hovered, pressed/active/down,
disabled/inactive. Per-state text colours of buttons are not generated (the default variant's text is used).

## Tags

| Tag | Effect |
|---|---|
| `[button]` | Treat as a button |
| `[focus]` | Default focus widget |
| `[9slice=16]`, `[9slice=l,t,r,b]` | 9-slice margins in px |
| `[action=IA_X]` | Reported, not generated |
| `[safezone]` | Wrap the screen in an `SSafeZone` |
| `[inline]` | Expand an instance instead of using the component class |
| `[ignore]` | Skip the layer |
| `[rasterize]` | Export as an image |
| `[widget=Border\|Image\|SizeBox\|Spacer\|Button]` | Force `SBorder`, `SImage`, `SBox`, `SSpacer` or `SButton` |

## Widget tree specs (contract between mapper, generator and preview)

Each generated class is an `FSlateHandoffWidgetTreeSpec` (`Kind` `Screen`, `Component`, `Button` or `Style`).
Widgets are listed parents first, siblings in draw order. `WidgetClass` is the Slate class name or a
generated user class. Property paths are Slate argument names; slot paths start with `Slot.`. **Values are
the C++ argument expressions exactly as generated; an empty value means "Slate default" and the argument is
not generated.** The planner diffs, stores and shows these strings.

### Value forms

| Form | Example |
|---|---|
| float | `12.f`, `0.5f`, `-3.25f` (4 decimals max) |
| bool | `true`, `false` |
| colour | `FLinearColor(0.2f, 0.4f, 1.f, 1.f)` |
| margin | `FMargin(12.f, 8.f, 12.f, 8.f)` (always four values when generated; the preview also reads `FMargin(u)` and `FMargin(h, v)`) |
| vector | `FVector2D(0.5f, 0.5f)`, `FVector4(8.f, 8.f, 8.f, 8.f)` |
| anchors | `FAnchors(0.f, 0.f, 1.f, 1.f)` |
| alignment | `HAlign_Fill`, `HAlign_Left`, `HAlign_Center`, `HAlign_Right`, `VAlign_Fill`, `VAlign_Top`, `VAlign_Center`, `VAlign_Bottom` |
| enum | `EVisibility::SelfHitTestInvisible`, `ETextJustify::Center`, `ETextTransformPolicy::ToUpper`, `ETextOverflowPolicy::Ellipsis`, `EWidgetClipping::ClipToBounds` |
| text | `NSLOCTEXT("Handoff", "SMainMenu.1a2b3c4d", "Play")`; inside a component: `InArgs._Text_Label` |
| font | `FHandoffUIStyle::Font("Inter", "Bold", 18.f)`, `FHandoffUIStyle::Font("Inter", "Bold", 18.f, 50)` |
| text style | `&FHandoffUIStyle::Get().GetWidgetStyle<FTextBlockStyle>("Text.Heading_H1")` |
| transform | `FSlateRenderTransform(FQuat2D(FMath::DegreesToRadians(15.f)))` |
| slot size | `Auto`, `Fill(1.f)` (generated as `.AutoWidth()` / `.FillWidth(1.f)` or the Height forms) |
| brush | `FSlateColorBrush(FLinearColor(...))`; `FSlateRoundedBoxBrush(FLinearColor(...), FVector4(8.f, 8.f, 8.f, 8.f), FLinearColor(...), 1.f)`; `FSlateRoundedBoxBrush(FLinearColor(...), FLinearColor(...), 1.f)`; `FSlateNoResource()`; a style brush: `*FHandoffUIStyle::Get().GetBrush("Image.T_Logo_f079c3.128x128")` (leading `*` = owned by the style set, passed as a pointer) |
| button style | `FButtonStyle().SetNormal(<brush>).SetHovered(<brush>).SetPressed(<brush>).SetDisabled(<brush>).SetNormalPadding(FMargin(...)).SetPressedPadding(FMargin(...))` |
| string (style entries) | `"Inter"`, `"Fonts/Inter-Bold.ttf"` |

### Properties per class

| Class | Paths |
|---|---|
| every widget | `Visibility`, `RenderOpacity`, `RenderTransform`, `RenderTransformPivot`, `Clipping`, `IsEnabled` |
| `STextBlock` | `Text`, `TextStyle`, `Font`, `ColorAndOpacity`, `Justification`, `AutoWrapText`, `LineHeightPercentage`, `TransformPolicy`, `OverflowPolicy`, `ShadowOffset`, `ShadowColorAndOpacity` |
| `SImage` | `Image` (brush), `ColorAndOpacity`, `DesiredSizeOverride` |
| `SBorder` | `BorderImage` (brush), `BorderBackgroundColor`, `Padding`, `HAlign`, `VAlign` |
| `SBox` | `WidthOverride`, `HeightOverride`, `HAlign`, `VAlign`, `Padding` |
| `SSpacer` | `Size` |
| `SWrapBox` | `InnerSlotPadding`, `PreferredSize`, `UseAllottedSize`, `HAlign` |
| `SButton` | `ButtonStyle` (button style), `ContentPadding`, `HAlign`, `VAlign` |
| `SConstraintCanvas`, `SOverlay`, `SHorizontalBox`, `SVerticalBox`, `SSafeZone` | common paths only |
| component class instance | `Text_<Layer>` arguments |

| Parent | Slot paths |
|---|---|
| `SConstraintCanvas` | `Slot.Anchors`, `Slot.Offset`, `Slot.Alignment`, `Slot.AutoSize` |
| `SHorizontalBox`, `SVerticalBox` | `Slot.Padding`, `Slot.Size`, `Slot.HAlign`, `Slot.VAlign` |
| `SWrapBox` | `Slot.Padding`, `Slot.HAlign`, `Slot.VAlign`, `Slot.FillEmptySpace` |
| `SOverlay` | `Slot.Padding`, `Slot.HAlign`, `Slot.VAlign` |
| `SBorder`, `SBox`, `SButton`, `SSafeZone` | none: content padding and alignment are the parent's own arguments |

Class-level values (`AssetProperties`): `@DesiredFocus` (`key:<widget key>` in the desired tree, the member
name once resolved) and `@Arg.Text_<Layer>` (default text of a component argument).

The style tree (`Kind` `Style`) has no hierarchy. Entries:

| Key | `WidgetClass` | Properties | Generated |
|---|---|---|---|
| `font:<Family>\|<Typeface>` | `FontFace` | `Family`, `Typeface`, `File` (string literals) | `AddFace("Inter", "Bold", TEXT("Fonts/Inter-Bold.ttf"));` |
| `text:<text style id>` | `FTextBlockStyle` | `Name` (string literal), `Font`, `ColorAndOpacity`, `ShadowOffset`, `ShadowColorAndOpacity`, `TransformPolicy` | `Style->Set("Text.X", FTextBlockStyle().SetFont(...)...);` |
| `brush:<name>` | `FSlateImageBrush` or `FSlateBoxBrush` | `Name` (`Image.<stem>.<W>x<H>`, `Box...`), `File` (resource-relative), `Size`, `Margin` (box only), `Tint` | `Style->Set("Image.T_Logo_f079c3.128x128", new FSlateImageBrush(FHandoffUIStyle::Resource(TEXT("Images/T_Logo_f079c3.png")), FVector2D(128.f, 128.f)));` |

Style entries are never removed automatically: an entry the package no longer has stays (classes generated
earlier may use it).

## Generated code

One argument per line, so a re-import merges your edits line by line (see `ReimportRules.md`). Widget
arguments come first in spec order, then slots (`Slot.Size` becomes `.AutoWidth()` / `.FillWidth(x)`, or the
`Height` forms in an `SVerticalBox`). Empty values are skipped. Helper wrappers use `SNew`, primary widgets
(`bIsVariable`) `SAssignNew` into a `TSharedPtr<Class>` member with a `/** "Layer" (TYPE) */` comment; every
widget has a `// "Layer" (TYPE)` or `// "Layer" [role]` comment above it.

Includes are computed from the classes and value expressions actually used. Includes of your own and
generated files come first and are relative to the including file, then the engine includes, each group sorted.
Colour and rounded-box brush values (`Image`, `BorderImage`) become an `FSlateBrush Brush_<widget>;` member assigned
at the top of `Construct` and passed as `.Image(&Brush_<widget>)`. A style brush value (`*Style::Get().GetBrush("Image.X")`)
is owned by the style set: no member, it is passed as `.Image(FHandoffUIStyle::Get().GetBrush("Image.X"))`. The
`ButtonStyle` value becomes an `FButtonStyle ButtonStyle_<widget>;` member assigned with one `.SetX(...)` per line
and passed as `.ButtonStyle(&...)`; style brushes inside it are dereferenced (`.SetNormal(*Style::Get().GetBrush("Image.X"))`).

`Generated/SMainMenuBase.h`:

```cpp
// Generated by Handoff for Slate from Figma frame "Main Menu" (1:2) in "Game UI".
// Do not edit by hand: the next import regenerates this file and merges your edits line by line.
// Put your code in SMainMenu.h / SMainMenu.cpp.

#pragma once

#include "CoreMinimal.h"
#include "Styling/SlateBrush.h"
#include "Widgets/DeclarativeSyntaxSupport.h"
#include "Widgets/SCompoundWidget.h"

class SConstraintCanvas;
class SPrimaryButton;
class STextBlock;

class SMainMenuBase : public SCompoundWidget
{
public:
	SLATE_BEGIN_ARGS(SMainMenuBase)
		{}
	SLATE_END_ARGS()

	void Construct(const FArguments& InArgs);

	/** Widget tagged [focus] in the design (null when none). */
	TSharedPtr<SWidget> GetDefaultFocusWidget() const;

protected:
	/** "Main Menu" (FRAME) */
	TSharedPtr<SConstraintCanvas> fig_0a1b2c3d;
	/** "Title" (TEXT) */
	TSharedPtr<STextBlock> fig_1a2b3c4d;
	/** "Play" (INSTANCE) */
	TSharedPtr<SPrimaryButton> fig_2b3c4d5e;

	FSlateBrush Brush_fig_0a1b2c3d_bg;
};
```

`Generated/SMainMenuBase.cpp`:

```cpp
// Generated by Handoff for Slate ... (same header comment)

#include "SMainMenuBase.h"

#include "../Components/SPrimaryButton.h"
#include "HandoffUIStyle.h"
#include "Brushes/SlateColorBrush.h"
#include "Widgets/Images/SImage.h"
#include "Widgets/Layout/SConstraintCanvas.h"
#include "Widgets/Text/STextBlock.h"

void SMainMenuBase::Construct(const FArguments& InArgs)
{
	Brush_fig_0a1b2c3d_bg = FSlateColorBrush(FLinearColor(0.01f, 0.01f, 0.02f, 1.f));

	ChildSlot
	[
		// "Main Menu" (FRAME)
		SAssignNew(fig_0a1b2c3d, SConstraintCanvas)
		.Visibility(EVisibility::SelfHitTestInvisible)
		+ SConstraintCanvas::Slot()
		.Anchors(FAnchors(0.f, 0.f, 1.f, 1.f))
		.Offset(FMargin(0.f, 0.f, 0.f, 0.f))
		[
			// "Main Menu" [bg]
			SNew(SImage)
			.Image(&Brush_fig_0a1b2c3d_bg)
			.Visibility(EVisibility::HitTestInvisible)
		]
		+ SConstraintCanvas::Slot()
		.Anchors(FAnchors(0.5f, 0.f, 0.5f, 0.f))
		.Offset(FMargin(-200.f, 120.f, 400.f, 48.f))
		[
			// "Title" (TEXT)
			SAssignNew(fig_1a2b3c4d, STextBlock)
			.Text(NSLOCTEXT("Handoff", "SMainMenu.5e6f7a8b", "MAIN MENU"))
			.TextStyle(&FHandoffUIStyle::Get().GetWidgetStyle<FTextBlockStyle>("Text.Heading_H1"))
			.Justification(ETextJustify::Center)
			.AutoWrapText(true)
			.Visibility(EVisibility::HitTestInvisible)
		]
	];
}

TSharedPtr<SWidget> SMainMenuBase::GetDefaultFocusWidget() const
{
	return fig_2b3c4d5e;
}
```

Members that were deleted in the design but are still used by your code are created after `ChildSlot` with
`SAssignNew(member, Class)` and `.Visibility(EVisibility::Collapsed)` plus a comment, so your code keeps
compiling; they are not in the layout.

`SMainMenu.h` / `SMainMenu.cpp` (yours, written once):

```cpp
#pragma once

#include "CoreMinimal.h"
#include "Generated/SMainMenuBase.h"

/** Screen "Main Menu". Generated once by Handoff for Slate; this file is yours and is never overwritten. */
class SMainMenu : public SMainMenuBase
{
public:
	/** Uses the generated arguments of SMainMenuBase, so they stay in sync with the design. */
	void Construct(const FArguments& InArgs);
};
```

```cpp
#include "SMainMenu.h"

void SMainMenu::Construct(const FArguments& InArgs)
{
	SMainMenuBase::Construct(InArgs);

	// Your code: bind events and data here, e.g.
	// fig_2b3c4d5e->SetOnClicked(FOnClicked::CreateSP(this, &SMainMenu::OnPlayClicked));
}
```

Button classes add `SLATE_EVENT(FOnClicked, OnClicked)` (passed to the root `SButton` with
`.OnClicked(InArgs._OnClicked)`), `void SetOnClicked(FOnClicked InOnClicked);` and a member for the root
`SButton`. Component classes declare their text arguments with the design text as default
(`: _Text_Label(NSLOCTEXT(...))`).

The style class (`Generated/HandoffUIStyle.h/.cpp`) is created and registered on first use:

```cpp
class FHandoffUIStyle
{
public:
	/** The style set (created and registered on first use). */
	static const ISlateStyle& Get();
	static FName GetStyleSetName();
	/** Unregisters the style set and releases fonts; call from your module's ShutdownModule (optional in the editor). */
	static void Shutdown();
	/** Absolute path of a file in the Handoff resources folder (Images/..., Fonts/...). */
	static FString Resource(const FString& RelativePath);
	/** Font of a design family and style in Slate points; the engine default font when the family is not mapped. */
	static FSlateFontInfo Font(FName Family, FName Typeface, float Size, int32 LetterSpacing = 0);
};
```

Fonts are loaded lazily from the resources folder as one composite font per family (`FStandaloneCompositeFont`,
one typeface per font file). `Font()` falls back to `FCoreStyle::GetDefaultFontStyle` (Regular, Bold, Italic,
BoldItalic or Light, chosen from the typeface name) when the family is not mapped, and sets the letter spacing
on the returned `FSlateFontInfo`. `Resource()` is built from `FPaths::ProjectDir()` for project modules and from
`IPluginManager::Get().FindPlugin(...)->GetBaseDir()` for plugin modules. Text styles are registered as
`Style->Set("Text.X", FTextBlockStyle().SetFont(...)...)` with one setter per line, empty values skipped. Image
brushes are registered as `Style->Set("Image.X", new FSlateImageBrush(Resource(...), Size))` (`FSlateBoxBrush` for
9-slice), sorted by name, and the style set's content root is the resources folder. `Get()` creates and registers
the set on first use; `Shutdown()` unregisters it and releases the fonts.
Packaged games must stage the resources folder (*Project Settings > Packaging > Additional Non-Asset
Directories to Package*, or `RuntimeDependencies` for a plugin module); **Check Setup** verifies it (HC05). See
`FontsAndStyles.md`.
