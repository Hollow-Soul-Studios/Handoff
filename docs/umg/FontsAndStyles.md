# Fonts and styles

**Nothing is bundled.** Handoff for UMG ships no font files. Fonts come from files you supply or from a Google
Fonts download you trigger yourself.

## How a font is resolved

For each family + style in the package (for example `Inter` + `SemiBold`), in this order:

1. A row in the **mapping table** (Project Settings > Plugins > Handoff for UMG > Fonts) pointing at an
   **existing composite font** (`UFont`) and typeface (default: the style name).
2. A mapping row pointing at a **TTF/OTF file**, absolute or relative to the project directory (if the file does
   not exist a warning is logged and the lookup continues).
3. A file in a **search folder** (default `<Project>/Fonts`; list in the same settings group) whose name,
   normalised, matches "FamilyStyle" (for example `Inter-SemiBold.ttf`, `Inter SemiBold.otf`). For the style
   Regular, a file named just like the family also matches.
4. A font face **imported by an earlier import** (reused).

If nothing matches the font is **missing**: it is flagged in the plan, the **Fonts** tab, the Messages tab and the
report (`NumMissingFonts`), and the text keeps the engine default font. The import is not blocked by it.
Line height is converted from the font file's metrics, so it depends on the font being found.

## Output

Under `<Destination>/Fonts/`:

| Asset | Content |
|---|---|
| `FF_<Family>_<Style>` | Inline `UFontFace` imported from the file |
| `F_<Family>` | Composite `UFont` with one typeface per Figma style name |
| `<Family>_OFL.txt` | License text, next to the font (Google Fonts downloads) |

Text widgets reference `F_<Family>` and the typeface for their style.

## Fonts tab

Per family/style status (imported, mapped file, found file, mapped asset, missing) and two actions:

* **Map file...** pick a TTF/OTF; a mapping row is added to the project settings.
* **Download Google Font (OFL)** fetches the family into `<Project>/Fonts/GoogleFonts/<Family>/` and adds the mapping
  rows.

## Google Fonts download

User-triggered only (button, or `unreal.HandoffLibrary.download_google_font(family, styles)`).

1. The importer checks `ofl/<family>/OFL.txt` in the `google/fonts` repository. If it is not there (the family is
   under another license) the download is refused.
2. Each requested style (default: Regular) is mapped to a Google Fonts weight; a style that cannot be mapped or does not
   exist is reported and skipped.
3. Files are saved as `<Project>/Fonts/GoogleFonts/<Family>/...` together with `OFL.txt`, and mapping rows are added.

You are responsible for complying with the font's license (OFL requires keeping the license with the font; the
importer keeps it for you next to the font assets). For commercial or proprietary fonts, map your own licensed
files instead.

## Text styles

Plain UMG: every text layer gets inline typography on a Text Block (font, size px x 0.75, letter spacing, line
height, colour).

CommonUI mode: for each design text style the importer generates a `UCommonTextStyle` Blueprint subclass
`Styles/TS_<TextStyle>`. A text layer uses `CommonTextBlock` + Style **only when it matches the style exactly**
(typography and colour); an override falls back to inline typography with an info message such as
"'Label' overrides text style 'Heading/H1' (typography or colour); imported with inline typography."

## Button styles (CommonUI mode)

For each button set: `Styles/BS_<Set>` (`UCommonButtonStyle`) with per-state brushes, ButtonPadding, MinWidth and
MinHeight, plus per-state text styles `Styles/TS_<Set>_<State>`. The button widget is a `UCommonButtonBase`
subclass; a Handoff-owned `HandoffApplyTextStyle` function bound to OnCurrentTextStyleChanged applies the state
text style. Plain UMG buttons use a `UButton` root with per-state `FButtonStyle` brushes.

## Colours

sRGB design colours are converted to linear. Display P3 documents are not converted (`source.colorProfile` is
informational), so colours can look slightly different from Figma on wide-gamut files.
