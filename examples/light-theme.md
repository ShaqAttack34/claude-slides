# Light Theme Starter

A ready-to-use light theme configuration. Copy these values into your `/Gslides` skill template.

## Colors

| Token | Hex | RGB (0-1) | Usage |
|-------|-----|-----------|-------|
| `bg` | `#FFFFFF` | `{red: 1, green: 1, blue: 1}` | Slide background |
| `text-primary` | `#1A1A1A` | `{red: 0.1, green: 0.1, blue: 0.1}` | Headings, key labels |
| `text-secondary` | `#666666` | `{red: 0.4, green: 0.4, blue: 0.4}` | Body text, descriptions |
| `text-muted` | `#999999` | `{red: 0.6, green: 0.6, blue: 0.6}` | Captions, page numbers |
| `accent` | `#[YOUR_BRAND_COLOR]` | Convert using: R/255, G/255, B/255 | Highlights, badges |
| `card-bg` | `#F5F5F5` | `{red: 0.96, green: 0.96, blue: 0.96}` | Card fill |
| `card-border` | `#E0E0E0` | `{red: 0.88, green: 0.88, blue: 0.88}` | Card borders |
| `highlight-bg` | Accent at ~8% opacity on white | Approximate as solid color | Highlighted cards |
| `highlight-border` | Same as `accent` | Same as `accent` | Highlighted card border |

## How to Calculate Highlight Background

For light themes, the "highlighted card" uses your accent color at ~8% opacity on white. Since the API doesn't support transparency, calculate the solid equivalent:

```
accent hex: #2563EB (example blue)
  R: 255 - (255 - 37) * 0.08 = 255 - 17 = 238  -> 238/255 = 0.93
  G: 255 - (255 - 99) * 0.08 = 255 - 12 = 243  -> 243/255 = 0.95
  B: 255 - (255 - 235) * 0.08 = 255 - 2 = 253   -> 253/255 = 0.99

highlight-bg: {red: 0.93, green: 0.95, blue: 0.99}
```

Formula: `255 - (255 - channel) * opacity`

Replace the accent hex with your brand color and recalculate.

## Font Recommendation

**Roboto** or **Open Sans** work well for light themes. Clean, professional, universally readable. Both available in Google Slides.

## Tips for Light Themes

- Card borders are optional on light themes (the fill contrast is usually enough), but they add polish.
- Avoid pure black (#000000) for text. Use #1A1A1A for a softer, more modern feel.
- Your accent color needs to work on both white backgrounds (text/badges) and as a solid fill (buttons/highlights). Test both.
- For numbered circles and bullet points, your accent color should have enough contrast against the white background. Avoid pastels.
