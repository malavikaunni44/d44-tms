# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Dev Server

The preview server uses macOS's built-in Apache (`/usr/sbin/httpd`) because the Claude Preview sandbox restricts child processes from reading files in the project directory. The workaround is to copy files to `/private/tmp/d44-tms-www/` first (Claude Code has full filesystem access to do this), then Apache serves from there.

**Before starting the server**, always run:
```bash
mkdir -p /private/tmp/d44-tms-www
cp /Users/bernardoramalho/Documents/claude-prototypes/d44-TMS/design-system/index.html /private/tmp/d44-tms-www/
cp /Users/bernardoramalho/Documents/claude-prototypes/d44-TMS/design-tokens.json /private/tmp/d44-tms-www/
```

Then start via `preview_start` with name `"Design System Docs"` (config in `.claude/launch.json`). The Apache config lives at `/private/tmp/d44-tms-httpd.conf` and must be regenerated if `/private/tmp` is cleared (see `.claude/httpd.conf` for the template).

---

## Icons

Always use Material Design Icons from Google Fonts for all icons. Never use emoji, unicode symbols, or any other icon library.

How to load them — add this to the `<head>` of every HTML file:
```html
<link href="https://fonts.googleapis.com/icon?family=Material+Icons+Outlined" rel="stylesheet">
```

How to use them in HTML:
```html
<span class="material-icons-outlined">icon_name</span>
```

Icon names must match exactly what's available at https://fonts.google.com/icons. When in doubt about an icon name, use a common one like `arrow_forward`, `close`, `menu`, `search`, `check`, etc.

---

## Repository Purpose

This is the **d44-TMS design system seed repository**. It holds the authoritative design tokens and style rules derived from the Manifest Material Design System in Figma. There is no application code here — this repo is the reference point for any UI work done in d44-TMS.

- **Token source of truth:** `design-tokens.json` (W3C Design Token Community Group format)
- **Figma source:** https://www.figma.com/design/Tulc1hjAI8hu7AeyGONCsq/Manifest-Material-Design-System
- **Remote:** https://github.com/bernardolcr/d44-TMS.git

There is no package.json, build system, or test runner — this is a documentation/data repository.

---

## Design System Rules

### Font

**Inter** exclusively. Import from Google Fonts if not locally available: `https://fonts.google.com/specimen/Inter`

### Colors

Always reference tokens from `design-tokens.json`. Never hardcode arbitrary values.

**Primary**
| Token | Value | Usage |
|---|---|---|
| `primary.main` | `#4f4cf3` | Primary buttons, key UI elements |
| `primary.hover` | `#3836d4` | Hover state |
| `primary.active` | `#16147f` | Pressed/active state |

**Backgrounds**
| Token | Value | Usage |
|---|---|---|
| `background.default` | `#ffffff` | Page and surface background |
| `background.secondary` | `#f3f4f6` | Secondary surfaces |
| `background.tertiary` | `#e5e7eb` | Card slots, dividers |
| `background.danger` | `#fef2f2` | Error state |
| `background.success` | `#ecfdf5` | Success state |
| `background.info` | `#e8ebff` | Info state |
| `background.warning` | `#fef3c7` | Warning state |
| `background.overlay` | `rgba(9, 21, 33, 0.6)` | Modal backdrop |

**Text & Icons**
| Token | Value | Usage |
|---|---|---|
| `text.primary` | `#030712` | Default body text |
| `text.secondary` | `#374151` | Secondary text |
| `text.tertiary` | `#6b7280` | Subdued/helper/placeholder text |
| `text.contrast` | `#ffffff` | Text on dark or primary backgrounds |
| `text.danger` | `#b91c1c` | Error messages |
| `text.success` | `#047857` | Success messages |
| `text.warning` | `#ba7500` | Warning messages |
| `text.disabled` | `#9ca3af` | Disabled state |

**Border:** `#e5e7eb` — cards, inputs, separators.

State colors (danger, success, warning, info) must always pair their `background.*` token with the matching `text.*` token.

---

### Typography

| Style | Size | Weight | Line Height | Letter Spacing |
|---|---|---|---|---|
| `display` | 32px | 700 | 42px | -0.03em |
| `heading` | 24px | 700 | 32px | -0.02em |
| `title` | 20px | 700 | 30px | -0.01em |
| `subtitle` | 16px | 700 | 24px | -0.01em |
| `body` | 16px | 400 | 24px | -0.01em |
| `subtextBold` | 14px | 600 | 20px | -0.01em |
| `subtext` | 14px | 400 | 20px | 0em |
| `captionBold` | 12px | 600 | 18px | 0em |
| `caption` | 12px | 400 | 18px | 0em |

---

### Spacing

4px base unit. Do not invent custom spacing values.

| Token | Value | | Token | Value |
|---|---|---|---|---|
| `spacing/1` | 4px | | `spacing/6` | 24px |
| `spacing/1.5` | 6px | | `spacing/7` | 28px |
| `spacing/2` | 8px | | `spacing/8` | 32px |
| `spacing/3` | 12px | | `spacing/9` | 36px |
| `spacing/4` | 16px | | `spacing/10` | 40px |
| `spacing/5` | 20px | | | |

---

### Border Radius

| Token | Value | Usage |
|---|---|---|
| `radius.xsmall` | 4px | Tags, small badges, icon buttons |
| `radius.small` | 8px | Buttons, inputs, chips |
| `radius.medium` | 16px | Cards, alerts, containers |
| `radius.large` | 24px | Dialogs, modals, overlays |
| `radius.full` | 9999px | Pills, toggles, fully rounded chips |

---

### Shadows

Shadows use brand indigo as the shadow color — not black or grey.

| Token | Value | Usage |
|---|---|---|
| `shadow.light.small` | `0px 4px 16px -8px rgba(56,54,212,0.16), 0px 0px 1px 0px rgba(56,54,212,0.5)` | Cards, dropdowns |
| `shadow.light.medium` | `0px 12px 16px -8px rgba(56,54,212,0.08), 0px 2px 2px 0px rgba(56,54,212,0.07), 0px -1px 2px 0px rgba(56,54,212,0.02)` | Dialogs, modals, popovers |

---

## Product Documentation

All product context documents live in `/docs`. Each file is named after the use case or concept it covers.

**Current documents:**
- `docs/execution-recovery-agents.md` — execution recovery agents use case

**Rules for reading docs:**
- Always read `docs/execution-recovery-agents.md` when building anything related to carrier recovery, missed pickups, disruption handling, or agent actions
- Only read documents that are relevant to what is being built in the current session
- As new documents are added to `/docs`, they will be listed here with a description of when to read them
