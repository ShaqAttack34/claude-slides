# Build On-Brand Slide Decks with Claude Code ; co-created by ClaudeCode, steered by Shareq Husain (Shaq)

Turn natural language into polished, on-brand Google Slides presentations. No design skills required.

**What you'll build:**
- A Google Slides MCP connection so Claude can create and edit presentations
- A `/Gslides` slash command baked with your brand's design system
- A master template deck your team can reference and duplicate

**What you need:**
- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) installed and running
- A Google Cloud project (free tier is fine)
- ~30 minutes for setup

**How it works once set up:**

```
You:     /Gslides quarterly board update with revenue metrics, hiring pipeline, and product roadmap
Claude:  *builds a 6-slide deck in your brand colors, opens it in your browser*
```

Every element is native Google Slides (text boxes, shapes, formatting). Fully editable by your team.

---

## Part 1: Set Up Google Slides MCP

MCP (Model Context Protocol) lets Claude talk to external services. You need the Google Slides MCP server.

### Step 1: Create a Google Cloud Project

1. Go to [console.cloud.google.com](https://console.cloud.google.com)
2. Create a new project (e.g., "Claude Slides")
3. Enable two APIs:
   - **Google Slides API** (search in the API Library)
   - **Google Drive API** (needed for file creation)

### Step 2: Create OAuth Credentials

1. Go to **APIs & Services > Credentials**
2. Click **Create Credentials > OAuth client ID**
3. Application type: **Web application**
4. Add an authorized redirect URI: `http://localhost:3000/oauth2callback`
5. Copy your **Client ID** and **Client Secret**

If you haven't configured the OAuth consent screen yet, do that first:
- User type: External (or Internal if using Google Workspace)
- Add scopes: `presentations`, `drive.file`
- Add yourself as a test user

### Step 3: Add the MCP Server to Claude Code

Open your Claude Code config file at `~/.claude.json` and add this to the `"mcpServers"` section:

```json
{
  "mcpServers": {
    "google-slides": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "mcp-google-slides"],
      "env": {
        "GOOGLE_CLIENT_ID": "[YOUR_CLIENT_ID]",
        "GOOGLE_CLIENT_SECRET": "[YOUR_CLIENT_SECRET]",
        "GOOGLE_REDIRECT_URI": "http://localhost:3000/oauth2callback"
      }
    }
  }
}
```

Replace `[YOUR_CLIENT_ID]` and `[YOUR_CLIENT_SECRET]` with the values from Step 2.

### Step 4: Authenticate

Restart Claude Code, then run:

```
npx mcp-google-slides auth
```

This opens a browser window for Google OAuth. Sign in and grant access.

**If the auth callback doesn't work** (common issue), you may need a simple local server to catch the redirect. Create a temporary file called `auth-server.js`:

```javascript
const http = require('http');
const url = require('url');

const server = http.createServer((req, res) => {
  const query = url.parse(req.url, true).query;
  if (query.code) {
    console.log('\nAuth code received! Copy this code:\n');
    console.log(query.code);
    console.log('\nPaste it into the auth prompt, then close this server (Ctrl+C).\n');
    res.end('Auth code received. You can close this tab.');
  } else {
    res.end('Waiting for auth redirect...');
  }
});

server.listen(3000, () => {
  console.log('Listening on http://localhost:3000/oauth2callback');
  console.log('Now run: npx mcp-google-slides auth');
});
```

Run it with `node auth-server.js`, then run the auth command in another terminal.

### Step 5: Verify

In Claude Code, type `/mcp`. You should see `google-slides` listed as connected. Try a quick test:

```
Create a Google Slides presentation called "Test Deck" with one slide
```

If Claude creates a presentation and returns a URL, you're good.

---

## Part 2: Define Your Brand

Before building the skill, map out your brand's design tokens. The Google Slides API uses **RGB values from 0 to 1** (not 0-255, not hex).

### Hex to RGB (0-1) Converter

Divide each hex component by 255:

```
#E8725C (coral)
  R: 0xE8 = 232 / 255 = 0.91
  G: 0x72 = 114 / 255 = 0.45
  B: 0x5C =  92 / 255 = 0.36

Result: {red: 0.91, green: 0.45, blue: 0.36}
```

### Fill In Your Brand Table

Copy this table and fill in your values:

```
BRAND DESIGN TOKENS
====================

Background:        hex: [______]  rgb: {red: [__], green: [__], blue: [__]}
Text primary:      hex: [______]  rgb: {red: [__], green: [__], blue: [__]}
Text secondary:    hex: [______]  rgb: {red: [__], green: [__], blue: [__]}
Text muted:        hex: [______]  rgb: {red: [__], green: [__], blue: [__]}
Accent:            hex: [______]  rgb: {red: [__], green: [__], blue: [__]}
Card fill:         hex: [______]  rgb: {red: [__], green: [__], blue: [__]}
Card border:       hex: [______]  rgb: {red: [__], green: [__], blue: [__]}
Highlight fill:    hex: [______]  rgb: {red: [__], green: [__], blue: [__]}
Highlight border:  hex: [______]  rgb: {red: [__], green: [__], blue: [__]}

Font family:       [____________]  (must be available in Google Slides)
```

### Theme Starters

**Dark theme** (black background, white text, colored accents):

| Token | Hex | RGB (0-1) |
|-------|-----|-----------|
| Background | `#000000` | `{red: 0, green: 0, blue: 0}` |
| Text primary | `#FFFFFF` | `{red: 1, green: 1, blue: 1}` |
| Text secondary | `#999999` | `{red: 0.6, green: 0.6, blue: 0.6}` |
| Text muted | `#666666` | `{red: 0.4, green: 0.4, blue: 0.4}` |
| Accent | `#[YOUR_BRAND_COLOR]` | Convert with formula above |
| Card fill | `#0A0A0A` | `{red: 0.04, green: 0.04, blue: 0.04}` |
| Card border | `#1F1F1F` | `{red: 0.12, green: 0.12, blue: 0.12}` |

**Light theme** (white background, dark text):

| Token | Hex | RGB (0-1) |
|-------|-----|-----------|
| Background | `#FFFFFF` | `{red: 1, green: 1, blue: 1}` |
| Text primary | `#1A1A1A` | `{red: 0.1, green: 0.1, blue: 0.1}` |
| Text secondary | `#666666` | `{red: 0.4, green: 0.4, blue: 0.4}` |
| Text muted | `#999999` | `{red: 0.6, green: 0.6, blue: 0.6}` |
| Accent | `#[YOUR_BRAND_COLOR]` | Convert with formula above |
| Card fill | `#F5F5F5` | `{red: 0.96, green: 0.96, blue: 0.96}` |
| Card border | `#E0E0E0` | `{red: 0.88, green: 0.88, blue: 0.88}` |

### Choose a Font

The font must be available in Google Slides. Safe choices:
- **Inter** (modern, clean, great for data)
- **Roboto** (Google's default, very readable)
- **Poppins** (friendly, rounded)
- **Montserrat** (bold, professional)
- **Open Sans** (neutral, safe)
- **Lato** (warm, humanist)

Pick one font family and use weight variations (bold, regular, light) for hierarchy.

---

## Part 3: Create the `/Gslides` Skill

The skill is a markdown file that Claude loads whenever you type `/Gslides`. It contains your entire design system so Claude builds everything on-brand without you repeating yourself.

### Create the File

```
.claude/commands/Gslides.md
```

### Full Skill Template

Copy the template below and replace all `[PLACEHOLDER]` values with your brand tokens from Part 2.

````markdown
---
description: Create an editable Google Slides presentation in [YOUR_COMPANY] style
---

## Your Role

You are a Google Slides presentation designer for [YOUR_COMPANY]. You create editable, shareable Google Slides in [YOUR_COMPANY]'s presentation style using the Google Slides MCP tools. Everything you build is native Google Slides elements, fully editable by the team.

## Canvas

Google Slides standard: **720 x 405 points** (16:9)

## Theme (RGB 0-1 for Google Slides API)

| Token | RGB (0-1) | Hex | Usage |
|-------|-----------|-----|-------|
| `bg` | `{red: [R], green: [G], blue: [B]}` | `[HEX]` | Slide background |
| `text-primary` | `{red: [R], green: [G], blue: [B]}` | `[HEX]` | Headings, key labels |
| `text-secondary` | `{red: [R], green: [G], blue: [B]}` | `[HEX]` | Body text, descriptions |
| `text-muted` | `{red: [R], green: [G], blue: [B]}` | `[HEX]` | Captions, page numbers |
| `accent` | `{red: [R], green: [G], blue: [B]}` | `[HEX]` | Highlights, badges, active borders |
| `card-bg` | `{red: [R], green: [G], blue: [B]}` | `[HEX]` | Card fill |
| `card-border` | `{red: [R], green: [G], blue: [B]}` | `[HEX]` | Card border, dividers |
| `highlight-bg` | `{red: [R], green: [G], blue: [B]}` | `[HEX]` | Highlighted card fill |
| `highlight-border` | `{red: [R], green: [G], blue: [B]}` | `[HEX]` | Highlighted card border |

## Typography Scale (Google Slides points)

| Element | Font | Size | Weight | Color |
|---------|------|------|--------|-------|
| Section label | [FONT] | 9pt | bold | `accent` |
| Slide title | [FONT] | 28pt | bold | `text-primary` |
| Subtitle | [FONT] | 12pt | regular | `text-secondary` |
| Card heading | [FONT] | 14-16pt | bold | `text-primary` |
| Card body | [FONT] | 10-11pt | regular | `text-secondary` |
| Badge text | [FONT] | 8-9pt | bold | `accent` |
| Big stat number | [FONT] | 28-36pt | bold | `accent` |
| Page number | [FONT] | 8pt | regular | `text-muted` |

## Standard Header Block

Every content slide uses this consistent header:

```
Section label:  x:30, y:5,   9pt bold [FONT], accent color
Slide title:    x:30, y:26,  28pt bold [FONT], text-primary
Subtitle:       x:30, y:58,  12pt [FONT], text-secondary
Content starts: y:85
```

Title/Cover and Closing slides use centered layouts instead.

## Component Recipes

### Standard Card (Rounded)
```
add_shape: ROUND_RECTANGLE
  x, y, width, height (positioned)

format_shape:
  fillColor: {rgbColor: [CARD-BG RGB]}
  outline: {color: {rgbColor: [CARD-BORDER RGB]}, weight: 1}
```

### Highlighted Card (Rounded)
```
add_shape: ROUND_RECTANGLE

format_shape:
  fillColor: {rgbColor: [HIGHLIGHT-BG RGB]}
  outline: {color: {rgbColor: [HIGHLIGHT-BORDER RGB]}, weight: 1}
```

### Section Label
```
add_multiple_text_boxes:
  text: "SECTION NAME"
  fontSize: 9, bold: true, fontFamily: "[FONT]"
  foregroundColor: {rgbColor: [ACCENT RGB]}
  alignment: "START"
```

### Slide Title
```
add_multiple_text_boxes:
  text: "Title Here"
  fontSize: 28, bold: true, fontFamily: "[FONT]"
  foregroundColor: {rgbColor: [TEXT-PRIMARY RGB]}
  alignment: "START"
```

### Subtitle
```
add_multiple_text_boxes:
  text: "Subtitle text here"
  fontSize: 12, fontFamily: "[FONT]"
  foregroundColor: {rgbColor: [TEXT-SECONDARY RGB]}
  alignment: "START"
```

### Numbered Circle
```
add_shape: ELLIPSE
  width: 22, height: 22
  text: "1", fontSize: 10, bold: true, fontFamily: "[FONT]"
  foregroundColor: {rgbColor: {red: 1, green: 1, blue: 1}}
  alignment: "CENTER"

format_shape:
  fillColor: {rgbColor: [ACCENT RGB]}
```

### Bullet Point
```
add_shape: ELLIPSE
  width: 8, height: 8

format_shape:
  fillColor: {rgbColor: [ACCENT RGB]}
```

### Divider Line (stays RECTANGLE, not rounded)
```
add_shape: RECTANGLE
  width: 660 (or section width), height: 2

format_shape:
  fillColor: {rgbColor: [CARD-BORDER RGB]}
```

### Badge/Pill (Rounded)
```
add_shape: ROUND_RECTANGLE
  width: 60, height: 16
  text: "LABEL", fontSize: 8, bold: true, fontFamily: "[FONT]"
  foregroundColor: {rgbColor: [ACCENT RGB]}
  alignment: "CENTER"

format_shape:
  fillColor: {rgbColor: [HIGHLIGHT-BG RGB]}
  outline: {color: {rgbColor: [ACCENT RGB]}, weight: 1}
```

### Page Number
```
add_multiple_text_boxes:
  text: "2" (slide number as string)
  fontSize: 8, fontFamily: "[FONT]"
  foregroundColor: {rgbColor: [TEXT-MUTED RGB]}
  alignment: "END"
  x: 680, y: 385, width: 20, height: 15
```
Add to every content slide. Skip on Title/Cover (slide 1) and Closing (last slide).

## Shape Usage Rules

| Shape | When to Use |
|-------|------------|
| `ROUND_RECTANGLE` | Cards, badges, highlighted boxes, metric containers |
| `RECTANGLE` | Divider lines, accent lines, progress bars |
| `ELLIPSE` | Numbered circles, bullet dots |

**Never use RECTANGLE for cards.** Always use ROUND_RECTANGLE for a modern, premium feel.

## Build Process

### Step 1: Create the Presentation
```
mcp__google-slides__create_presentation
  title: "[Description] -- [YOUR_COMPANY]"
```
Save the returned presentationId.

### Step 2: Get First Slide ID
```
mcp__google-slides__get_presentation
  presentationId: [from step 1]
```

### Step 3: Set Background
```
mcp__google-slides__set_slide_background
  presentationId: [id]
  slideId: [first slide objectId]
  backgroundColor: {rgbColor: [BG RGB]}
```

### Step 4: Build Content
1. Background cards/shapes first (they layer behind)
2. Text boxes on top
3. Accent elements (bullets, badges) last
4. Page number last (bottom-right)

**Layout tips:**
- Left margin: 30pt
- Header: label y:5, title y:26, subtitle y:58
- Content starts: y:85
- Card padding: ~15pt internal
- Gap between cards: ~10pt
- Full-width content: 660pt wide (30pt margins each side)
- Page number: x:680, y:385 (skip on first and last slides)

### Step 5: Additional Slides
```
mcp__google-slides__create_slide
  presentationId: [id]
```
Repeat Steps 3-4 for each new slide.

### Step 6: Return the URL
```bash
open "https://docs.google.com/presentation/d/[presentationId]/edit"
```

## Known Gotchas

- **Empty text strings crash**: NEVER use `text:""` in text boxes. The API throws "object has no text" when styling. Always use actual text or skip the text box.
- **Outline weight must be >= 1**: `weight: 0` causes an error. Use `weight: 1` or omit outline entirely.
- **`set_element_position` has y:20 floor**: The MCP clamps y values to minimum 20pt. To place elements above y:20, use `batch_update` with `updatePageElementTransform` (ABSOLUTE mode, translateY in EMU: 1pt = 12700 EMU).
- **No configurable corner radius**: ROUND_RECTANGLE uses Google's default. Looks clean but you can't adjust it.
- **No gradients**: Shapes only support solid fills.
- **No rgba transparency**: Approximate translucent colors as solid dark/light variants.
- **Text wrapping**: Text boxes auto-wrap. Set widths carefully to control line breaks.
- **`get_slide` strips text**: Returns shapes and transforms but no text content. Use `get_thumbnail` to verify slides visually.

## User Input

The user's request: $ARGUMENTS

Now create the Google Slides presentation. Build it natively with ROUND_RECTANGLE shapes and text boxes for maximum editability. Set the background color, use the brand colors, use the standard header block, add page numbers, and aim for a clean professional layout. Open the Google Slides URL in the browser when done.
````

---

## Part 4: Build a Master Template Deck

A master template deck serves two purposes:
1. **Reference for Claude**: you can point future prompts at it
2. **Style guide for your team**: everyone can see what on-brand slides look like

### Recommended Slide Types

Build 10-15 template slides covering your most common needs:

| # | Slide Type | Good For |
|---|-----------|----------|
| 1 | **Title / Cover** | Opening slide with company name, deck title, date |
| 2 | **Metrics Dashboard** | 3-6 KPI cards with numbers, trends, sparklines |
| 3 | **Unit Economics** | Revenue, costs, margins in card layout |
| 4 | **Strategy / Big Bet** | One key initiative with context and rationale |
| 5 | **Board Update** | Status summary with RAG indicators |
| 6 | **Project Update** | Timeline, status, blockers, next steps |
| 7 | **Roadmap / Timeline** | Horizontal timeline with milestones |
| 8 | **Architecture / Technical** | System diagram or decision record |
| 9 | **User Research** | Persona cards, quotes, insights |
| 10 | **Hiring Pipeline** | Funnel stages with counts |
| 11 | **Team Health** | Survey results, engagement metrics |
| 12 | **Agenda / Process** | Meeting agenda or process steps |
| 13 | **Comparison** | Side-by-side options or before/after |
| 14 | **Color Palette** | Visual swatches of all brand colors with hex codes |
| 15 | **Closing / CTA** | Call to action, next steps, company tagline |

Pick the ones relevant to your team. You don't need all 15.

### Build It

```
/Gslides build a master template deck with 12 slides:
Title, Metrics Dashboard, Strategy, Project Update, Roadmap,
Comparison, Team Health, Agenda, Hiring Pipeline, Color Palette,
and Closing. Use placeholder content that demonstrates each layout.
```

Claude will create the deck and open it in your browser. Review it, request tweaks, iterate until it matches your brand.

### Save the Reference

Once you're happy, add the deck URL to the bottom of your `/Gslides` skill file:

```markdown
## Reference

Master template deck: `https://docs.google.com/presentation/d/[YOUR_DECK_ID]/edit`
```

This gives Claude a reference point for future decks.

---

## Part 5: Generate Decks On Demand

Once set up, generating decks is a one-liner.

### Basic Usage

```
/Gslides [describe what you need]
```

### Example Prompts

```
/Gslides quarterly board update: revenue hit $9.1M (up 34%),
3 new enterprise clients, hiring 4 engineers in Q2,
key risk is churn in SMB segment

/Gslides product roadmap for Q2:
3 major features (AI search, billing v2, mobile app),
timeline from April to June, team allocation

/Gslides investor pitch: problem (home services are fragmented),
solution (AI-powered platform), traction (50K users, $10.2M ARR),
team (4 founders), ask ($25M Series B)

/Gslides team all-hands: celebrate Q1 wins, introduce new hires,
preview Q2 priorities, open Q&A
```

### Tips for Best Results

1. **Give context, not instructions**: describe WHAT you're presenting, not HOW to build slides. Claude knows the design system.
2. **Include real numbers**: "revenue grew 34% to $9.1M" produces better slides than "show revenue growth".
3. **Specify the audience**: "board update" vs "team all-hands" changes the tone and detail level.
4. **List key points**: bullet points in your prompt map naturally to slide content.
5. **Iterate**: after the first build, ask Claude to adjust specific slides. "Make slide 3 more visual" or "add a timeline to slide 5".

---

## Gotchas and Tips

Hard-won lessons from lots of slides with this workflow.

### API Gotchas

| Issue | Cause | Fix |
|-------|-------|-----|
| "Object has no text" error | Empty string `""` passed to a text box | Always use actual text or skip the text box |
| "Invalid outline weight" | `weight: 0` in outline | Use `weight: 1` minimum or omit outline |
| Element won't go above y:20 | `set_element_position` clamps y minimum | Use `batch_update` with `updatePageElementTransform` (ABSOLUTE mode, translateY in EMU) |
| Can't read slide text | `get_slide` returns shapes but strips text | Use `get_thumbnail` with size `LARGE` to verify visually |
| Auth token expired | OAuth tokens expire after ~1 hour | Re-run `npx mcp-google-slides auth` or use refresh token |
| MCP won't connect | Server crashed or token expired | Run `/mcp` in Claude Code, restart if needed |

### Design Tips

- **ROUND_RECTANGLE for everything** except thin divider lines. Rounded corners look more modern and premium.
- **One font family, multiple weights**: use bold for headings, regular for body, light for captions. Don't mix font families.
- **Consistent header block**: same position on every slide creates visual rhythm. Section label at top, title below, subtitle below that.
- **Page numbers on content slides**: skip the title slide and closing slide.
- **Dark themes need card borders**: without borders, dark cards disappear into dark backgrounds. Use a subtle border (e.g., #1F1F1F on #0A0A0A).
- **Build order matters**: create background shapes first, then text on top, then accent elements last. Google Slides layers elements in creation order.

### EMU Reference

The Google Slides API uses EMU (English Metric Units) internally. When using `batch_update`, you'll need these conversions:

```
1 point = 12700 EMU
1 inch  = 914400 EMU

Slide width:  720pt = 9,144,000 EMU
Slide height: 405pt = 5,143,500 EMU

Common positions:
  x: 30pt  = 381,000 EMU    (left margin)
  y: 5pt   = 63,500 EMU     (section label)
  y: 26pt  = 330,200 EMU    (title)
  y: 85pt  = 1,079,500 EMU  (content start)
  y: 385pt = 4,889,500 EMU  (page number)
```

---

## Quick Reference Card

```
SETUP
  1. Google Cloud project + enable Slides API + Drive API
  2. OAuth credentials (Web app, redirect: localhost:3000/oauth2callback)
  3. Add MCP config to ~/.claude.json
  4. Run: npx mcp-google-slides auth
  5. Verify: /mcp in Claude Code

SKILL FILE
  .claude/commands/Gslides.md
  Contains: brand colors (RGB 0-1), font, layout positions, component recipes

USAGE
  /Gslides [describe your deck]

CANVAS
  720 x 405 points (16:9)

HEADER BLOCK
  Section label: x:30, y:5 (9pt bold, accent)
  Title:         x:30, y:26 (28pt bold, primary)
  Subtitle:      x:30, y:58 (12pt, secondary)
  Content:       y:85

SHAPES
  Cards/badges:  ROUND_RECTANGLE
  Divider lines: RECTANGLE
  Bullets/dots:  ELLIPSE

PAGE NUMBER
  x:680, y:385, 8pt, muted color, right-aligned
  Skip on first and last slides
```

---

Built by Shaq who got tired of making slides by hand. Now Claude does it in seconds, on-brand every time.
