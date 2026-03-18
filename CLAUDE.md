# p44-TMS Design System

This project uses the **Manifest Material Design System**. Always refer to `design-tokens.json` for the authoritative token values. The rules below summarize how to apply them.

- Figma source: https://www.figma.com/design/Tulc1hjAI8hu7AeyGONCsq/Manifest-Material-Design-System
- Font: **Inter** (always import from Google Fonts if not available locally)
- Base spacing unit: **4px**

---

## Colors

### Primary
| Token | Value | Usage |
|---|---|---|
| primary.main | `#4f4cf3` | Primary buttons, key UI elements |
| primary.hover | `#3836d4` | Hover state on primary elements |
| primary.active | `#16147f` | Pressed/active state |

### Backgrounds
| Token | Value | Usage |
|---|---|---|
| background.default | `#ffffff` | Page and surface background |
| background.secondary | `#f3f4f6` | Secondary surfaces |
| background.tertiary | `#e5e7eb` | Cards slots, dividers |
| background.danger | `#fef2f2` | Error state background |
| background.success | `#ecfdf5` | Success state background |
| background.info | `#e8ebff` | Info state background |
| background.warning | `#fef3c7` | Warning state background |
| background.overlay | `rgba(9, 21, 33, 0.6)` | Modal backdrop |

### Text & Icons
| Token | Value | Usage |
|---|---|---|
| text.primary | `#030712` | Default body text |
| text.secondary | `#374151` | Secondary text |
| text.tertiary | `#6b7280` | Subdued/helper text |
| text.contrast | `#ffffff` | Text on dark or primary backgrounds |
| text.danger | `#b91c1c` | Error messages |
| text.success | `#047857` | Success messages |
| text.warning | `#ba7500` | Warning messages |
| text.disabled | `#9ca3af` | Disabled state |

### Borders
- Default border: `#e5e7eb` — use for cards, inputs, and separators

---

## Typography

All text uses **Inter**. Never substitute another font.

| Style | Size | Weight | Line Height | Letter Spacing | Usage |
|---|---|---|---|---|---|
| display | 32px | 700 | 42px | -0.03em | Page heroes, large headings |
| heading | 24px | 700 | 32px | -0.02em | Section headings, card headers |
| title | 20px | 700 | 30px | -0.01em | Dialog titles, modal headers |
| subtitle | 16px | 700 | 24px | -0.01em | Emphasized body, section labels |
| body | 16px | 400 | 24px | -0.01em | Default body copy |
| subtextBold | 14px | 600 | 20px | -0.01em | Button labels, emphasized secondary |
| subtext | 14px | 400 | 20px | 0em | Secondary copy, helper text |
| captionBold | 12px | 600 | 18px | 0em | Emphasized labels |
| caption | 12px | 400 | 18px | 0em | Metadata, smallest labels |

---

## Spacing

Based on a **4px base unit**. Use these values — do not invent custom spacing.

| Token | Value |
|---|---|
| spacing/1 | 4px |
| spacing/1.5 | 6px |
| spacing/2 | 8px |
| spacing/3 | 12px |
| spacing/4 | 16px |
| spacing/5 | 20px |
| spacing/6 | 24px |
| spacing/7 | 28px |
| spacing/8 | 32px |
| spacing/9 | 36px |
| spacing/10 | 40px |

---

## Border Radius

| Token | Value | Usage |
|---|---|---|
| xsmall | 4px | Tags, small badges, icon buttons |
| small | 8px | Buttons, inputs, chips |
| medium | 16px | Cards, alerts, containers |
| large | 24px | Dialogs, modals, overlays |
| full | 9999px | Pills, toggles, fully rounded chips |

---

## Shadows

Shadows use the brand indigo as the shadow color — not black.

| Token | Value | Usage |
|---|---|---|
| shadow.light.small | `0px 4px 16px -8px rgba(56,54,212,0.16), 0px 0px 1px 0px rgba(56,54,212,0.5)` | Cards, dropdowns |
| shadow.light.medium | `0px 12px 16px -8px rgba(56,54,212,0.08), 0px 2px 2px 0px rgba(56,54,212,0.07), 0px -1px 2px 0px rgba(56,54,212,0.02)` | Dialogs, modals, popovers |

---

## Rules

- **Always** use token values from this file or `design-tokens.json`. Never hardcode arbitrary colors or spacing.
- **Always** use Inter. Import it from Google Fonts if needed: `https://fonts.google.com/specimen/Inter`
- **Always** use the correct border radius for the component type (e.g. `small` for buttons, `medium` for cards).
- **Always** use indigo-tinted shadows, never plain black/grey shadows.
- Use `text.tertiary` for placeholder and helper text, never grey arbitrary values.
- State colors (danger, success, warning, info) must always use both the background token AND the matching text token together.
- When building interactive components, always include hover, active, and disabled states using the appropriate tokens.
