# Fonts, styles and images

**Nothing is bundled.** Handoff for Slate ships no font files. Fonts come from files you supply or from a
Google Fonts download you trigger yourself. Fonts and images are plain files copied into the **resources
folder** (default `Content/Slate/Handoff`) and loaded at runtime by the generated style class.

## How a font is resolved

For each family + style in the package (for example `Inter` + `SemiBold`), in this order:

1. A row in the **mapping table** (Project Settings > Plugins > Handoff for Slate > Fonts) pointing at a
   **TTF/OTF file**, absolute or relative to the project directory (if the file does not exist a warning is
   logged and the lookup continues).
2. A file in a **search folder** (default `<Project>/Fonts`, searched recursively; list in the same settings
   group) whose name, normalised, matches "FamilyStyle" (for example `Inter-SemiBold.ttf`,
   `Inter SemiBold.otf`). For the style Regular, a file named just like the family also matches.
3. A font **copied by an earlier import** into the resources folder (reused).

If nothing matches the font is **missing**: it is flagged in the plan, the **Fonts** tab, the Messages tab and
the report (`NumMissingFonts`), the generated code still names the font, and at runtime the text uses the engine
default font ("Font 'Inter Bold' is not mapped. Text using it uses the engine default font at runtime until you
map a TTF/OTF file..."). The import is not blocked by it. Mapping the font later and re-importing makes the text
pick it up; the generated code does not change, only the file in the resources folder appears. Line height is
converted from the font file's metrics (hhea, or OS/2 typo metrics when the font asks for them), so it
depends on the font being found; 1.2 is assumed otherwise.

## Output

Under `<Resources folder>/Fonts/`:

| File | Content |
|---|---|
| `<Family>-<Style>.ttf` (or `.otf`) | The font file, copied from your mapped or found file; the name is PascalCased from the family and style |
| `<Family>-OFL.txt` | License text, next to the font (when the source folder holds an `OFL.txt`, as for Google Fonts downloads) |

The generated style class registers each file as one typeface of a composite font per family:
`AddFace("Inter", "Bold", TEXT("Fonts/Inter-Bold.ttf"))`. Text widgets get their font from
`FHandoffUIStyle::Font("Inter", "Bold", 18.f)`; with a family that has no registered typeface this returns the
engine default font (Regular, Bold, Italic, BoldItalic or Light, chosen from the style name). Fonts load lazily
on first use.

## Fonts tab

Per family/style status (copied earlier, mapped file, found file, missing) and two actions:

* **Map file...** pick a TTF/OTF; a mapping row is added to the project settings.
* **Download Google Font (OFL)** fetches the family into `<Project>/Fonts/GoogleFonts/<Family>/` and adds the
  mapping rows.

## Google Fonts download

User-triggered only (button, or `unreal.SlateHandoffLibrary.download_google_font(family, styles)`).

1. The importer checks `ofl/<family>/OFL.txt` in the `google/fonts` repository. If it is not there (the family is
   under another license) the download is refused.
2. Each requested style (default: Regular) is mapped to a Google Fonts weight; a style that cannot be mapped or does not
   exist is reported and skipped.
3. Files are saved as `<Project>/Fonts/GoogleFonts/<Family>/<Family>-<Style>.ttf` together with `OFL.txt`, and
   mapping rows are added. The next import copies the files and the license into the resources folder.

You are responsible for complying with the font's license (OFL requires keeping the license with the font; the
importer keeps it for you next to the font files). For commercial or proprietary fonts, map your own licensed
files instead.

## Text styles

Every text layer is a `STextBlock`. When it matches a design text style exactly (typography and colour) it uses
`.TextStyle(&FHandoffUIStyle::Get().GetWidgetStyle<FTextBlockStyle>("Text.Heading_H1"))`; the style class holds
one `FTextBlockStyle` entry per design text style, registered as `Text.<Identifier(style name)>`. A text layer
that differs from its style falls back to inline typography (`Font`, `ColorAndOpacity`, shadow, transform
policy) with an info message such as "'Label' overrides text style 'Heading/H1' (typography or colour); imported
with inline typography." Font size is px x 0.75 in Slate points; letter spacing is per-mille.

## Button styles

For each button set the generated class holds an `FButtonStyle` member with Normal / Hovered / Pressed /
Disabled brushes taken from the variants and padding from the design. See [CodeGeneration.md](CodeGeneration.md).
Per-state text colours are not generated.

## Image brushes

Slate loads image files only for the brushes of a registered style set, so images are **style entries**: the
style class registers `Image.<file stem>.<W>x<H>` (`FSlateImageBrush`) and `Box.<file stem>.<W>x<H>.<4 hex of the
margins>` (`FSlateBoxBrush`, for `[9slice]`), with `.Tint<percent>` when the layer has reduced opacity, and the
widgets reference them with `FHandoffUIStyle::Get().GetBrush("Image.T_Logo_f079c3.128x128")`. The files are
`<Resources folder>/Images/T_<Name>_<hash6>.png` (the 1x or 2x PNG according to *Textures*; the brush size stays
at 1x).

## Packaging

The resources folder is not a UAsset, so a cooked game does not contain it unless you stage it.

* **Project module** (resources folder below `Content`, the default `Content/Slate/Handoff`): add the path
  relative to `Content` (`Slate/Handoff`) to *Project Settings > Packaging > Additional Non-Asset Directories to
  Package*.
* **Plugin module** (resources folder below the plugin, default `Resources/Handoff` of the plugin): declare it in
  the module's `Build.cs`: `RuntimeDependencies.Add("$(PluginDir)/Resources/Handoff/...", StagedFileType.NonUFS);`
  and list `/Resources/Handoff/...` in the plugin's `Config/FilterPlugin.ini` when you distribute the plugin.
* A resources folder outside `Content` (project) or outside the plugin cannot be staged and is a failed check.

`Style::Resource()` resolves relative to `FPaths::ProjectDir()` (project module) or the plugin's base directory
(plugin module). **Check Setup** (HC05, "Resources staged for packaging") reports what is missing and the line to
add. A packaged game without the staged files shows white or missing images and the default font; see
[Troubleshooting.md](Troubleshooting.md).

## Colours

sRGB design colours are converted to linear. Display P3 documents are not converted (`source.colorProfile` is
informational), so colours can look slightly different from Figma on wide-gamut files.
