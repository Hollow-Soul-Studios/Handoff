# Handoff documentation

Handoff imports a design package into Unreal Engine and re-imports after design changes without losing your own work.
Choose your product:

* **Handoff for UMG** generates Widget Blueprints (optionally CommonUI). Start at the
  [UMG plugin README](umg/plugin-readme.md).
* **Handoff for Slate** generates Slate C++ classes with a generation gap and line-level merge. Start at the
  [Slate plugin README](slate/plugin-readme.md).

Both work with Figma design through the same free companion exporter ([exporter guide](figma-exporter.md)) and the same
tool-neutral package format. The package contract is the layout schema: [LayoutSchema (UMG copy)](umg/LayoutSchema.md);
the [Slate copy](slate/LayoutSchema.md) describes the same `layout.json` schema v1.

The pages marked *synced* are copied from each plugin's own `Docs/` folder by the release pipeline, so they always match
the shipped version.

## Shared

| Page | Contents |
|---|---|
| [figma-exporter.md](figma-exporter.md) | Installing and using the companion Figma exporter (both products) |
| [../SUPPORT.md](../SUPPORT.md) | How to file an issue |
| [../CHANGELOG.md](../CHANGELOG.md) | Release notes, per product |
| [../ROADMAP.md](../ROADMAP.md) | What is planned |

## Handoff for UMG

### Getting started

| Page | Contents |
|---|---|
| [umg/plugin-readme.md](umg/plugin-readme.md) | *Synced.* Overview, requirements, install, 5-minute quick start, known limits |
| [umg/QuickStart.md](umg/QuickStart.md) | *Synced.* Export, import, review, re-import, restore |
| [umg/SCENARIO.md](umg/SCENARIO.md) | *Synced.* Sample packages and the v1 to v2 re-import scenario |

### Reference

| Page | Contents |
|---|---|
| [umg/FeatureMapping.md](umg/FeatureMapping.md) | *Synced.* Design element to UMG output, plain and CommonUI mode, layer tags |
| [umg/ReimportRules.md](umg/ReimportRules.md) | *Synced.* Identity, re-link, 3-way merge, removals, safety net, backup and undo caveat |
| [umg/FontsAndStyles.md](umg/FontsAndStyles.md) | *Synced.* Font mapping, Google Fonts (OFL), text and CommonUI styles |
| [umg/Automation.md](umg/Automation.md) | *Synced.* Python, commandlet, CI exit codes, `-Explain`, `-Render`, console commands, REST fetch |
| [umg/LayoutSchema.md](umg/LayoutSchema.md) | *Synced.* Package format (`layout.json` schema v1) for exporter authors; shared by both products |
| [umg/DesignLog.md](umg/DesignLog.md) | *Synced.* Dated clean-room design log |

### Help

| Page | Contents |
|---|---|
| [umg/Troubleshooting.md](umg/Troubleshooting.md) | *Synced.* Symptom, cause, fix |
| [umg/Support.md](umg/Support.md) | *Synced.* Health checks, support bundles and filing an issue (as shipped with the plugin) |
| [umg/support/health-checks.md](umg/support/health-checks.md) | The 16 setup checks (HC01 to HC16) and how to fix them |
| [umg/support/support-bundle.md](umg/support/support-bundle.md) | What the support bundle contains and how to inspect it |
| [umg/support/faq.md](umg/support/faq.md) | Frequently asked questions |

## Handoff for Slate

### Getting started

| Page | Contents |
|---|---|
| [slate/plugin-readme.md](slate/plugin-readme.md) | *Synced.* Overview, requirements, install, 5-minute quick start, known limits |
| [slate/QuickStart.md](slate/QuickStart.md) | *Synced.* Export, import, review, re-import, restore |
| [slate/SCENARIO.md](slate/SCENARIO.md) | *Synced.* Sample packages and the v1 to v2 re-import scenario |

### Reference

| Page | Contents |
|---|---|
| [slate/CodeGeneration.md](slate/CodeGeneration.md) | *Synced.* Design element to Slate code, files and names, layer tags, the style class |
| [slate/ReimportRules.md](slate/ReimportRules.md) | *Synced.* Identity, re-link, line-level 3-way merge, locks, safety net, backup and undo caveat |
| [slate/FontsAndStyles.md](slate/FontsAndStyles.md) | *Synced.* Font mapping, Google Fonts (OFL), the style class and text styles |
| [slate/Automation.md](slate/Automation.md) | *Synced.* Python, commandlet, CI exit codes, `-Explain`, `-Render`, console commands, REST fetch |
| [slate/LayoutSchema.md](slate/LayoutSchema.md) | *Synced.* Package format (`layout.json` schema v1); same contract as the UMG copy |
| [slate/DesignLog.md](slate/DesignLog.md) | *Synced.* Dated clean-room design log |

### Help

| Page | Contents |
|---|---|
| [slate/Troubleshooting.md](slate/Troubleshooting.md) | *Synced.* Symptom, cause, fix |
| [slate/Support.md](slate/Support.md) | *Synced.* Health checks, support bundles and filing an issue (as shipped with the plugin) |
| [slate/support/health-checks.md](slate/support/health-checks.md) | The 16 setup checks (HC01 to HC16) and how to fix them |
| [slate/support/support-bundle.md](slate/support/support-bundle.md) | What the support bundle contains and how to inspect it |
| [slate/support/faq.md](slate/support/faq.md) | Frequently asked questions |

*Figma is a trademark of Figma, Inc. Handoff for UMG and Handoff for Slate are independent products and are not
affiliated with or endorsed by Figma.*
