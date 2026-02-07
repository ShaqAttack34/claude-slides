# Dark Theme Starter

A ready-to-use dark theme configuration. Copy these values into your `/Gslides` skill template.

## Colors

| Token | Hex | RGB (0-1) | Usage |
|-------|-----|-----------|-------|
| `bg` | `#000000` | `{red: 0, green: 0, blue: 0}` | Slide background |
| `text-primary` | `#FFFFFF` | `{red: 1, green: 1, blue: 1}` | Headings, key labels |
| `text-secondary` | `#999999` | `{red: 0.6, green: 0.6, blue: 0.6}` | Body text, descriptions |
| `text-muted` | `#666666` | `{red: 0.4, green: 0.4, blue: 0.4}` | Captions, page numbers |
| `accent` | `#[YOUR_BRAND_COLOR]` | Convert using: R/255, G/255, B/255 | Highlights, badges |
| `card-bg` | `#0A0A0A` | `{red: 0.04, green: 0.04, blue: 0.04}` | Card fill |
| `card-border` | `#1F1F1F` | `{red: 0.12, green: 0.12, blue: 0.12}` | Card borders |
| `highlight-bg` | Accent at ~10% opacity on black | Approximate as solid color | Highlighted cards |
| `highlight-border` | Same as `accent` | Same as `accent` | Highlighted card border |

## How to Calculate Highlight Background

For dark themes, the "highlighted card" uses your accent color at ~10% opacity on black. Since the API doesn't support transparency, calculate the solid equivalent:

```
accent hex: #E8725C (example)
  R: 232 * 0.10 = 23  -> 23/255 = 0.09
  G: 114 * 0.10 = 11  -> 11/255 = 0.04
  B:  92 * 0.10 =  9  ->  9/255 = 0.04

highlight-bg: {red: 0.09, green: 0.04, blue: 0.04}
```

Replace the accent hex with your brand color and recalculate.

## Font Recommendation

**Inter** works exceptionally well for dark themes. Clean, modern, highly readable at small sizes. Available in Google Slides.

## Tips for Dark Themes

- Always add a subtle border to cards (`card-border`). Without it, dark cards vanish into the black background.
- Use `text-muted` (#666666) sparingly. On black backgrounds, grey text below #666 becomes hard to read.
- For the `bg` swatch on your color palette slide, add a visible border so it doesn't disappear.
- White text on black is high contrast. Consider using `#F0F0F0` (`{red: 0.94, green: 0.94, blue: 0.94}`) instead of pure white if it feels harsh.
