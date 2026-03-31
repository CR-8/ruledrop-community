Apply a cohesive visual theme — colors, typography, and spacing — to any artifact: HTML pages, slide decks, dashboards, reports, or UI components. Use this skill when the user wants to style or restyle an existing artifact, or when building something new that needs a strong visual identity.

## How to Apply a Theme

1. Ask the user which theme they want, or show them the options below
2. Wait for their choice (or generate a custom one if none fit)
3. Apply the theme's colors, fonts, and spacing consistently throughout the artifact

## Available Themes

### 1. Obsidian
Dark, minimal, high-contrast. For technical tools and developer-facing UIs.
- Background: `#0d0d0d` | Surface: `#1a1a1a` | Border: `#2a2a2a`
- Accent: `#e2ff5d` (electric lime) | Text: `#f0f0f0` | Muted: `#666`
- Fonts: `JetBrains Mono` (headings) + `Inter` (body — exception here for readability)
- Vibe: terminal, precise, no decoration

### 2. Ivory & Ink
Light, editorial, typographic. For documentation, blogs, and content-heavy pages.
- Background: `#faf8f4` | Surface: `#f0ece4` | Border: `#ddd8ce`
- Accent: `#c0392b` (deep red) | Text: `#1a1a1a` | Muted: `#888`
- Fonts: `Playfair Display` (headings) + `Source Serif 4` (body)
- Vibe: newspaper, refined, literary

### 3. Aurora
Dark with vibrant gradients. For creative portfolios, landing pages, and marketing.
- Background: `#07080f` | Surface: `#0f1120`
- Gradient: `135deg, #6c63ff → #ff6584 → #43e97b`
- Text: `#ffffff` | Muted: `rgba(255,255,255,0.5)`
- Fonts: `Syne` (headings) + `DM Sans` (body)
- Vibe: cosmic, energetic, modern

### 4. Sand & Clay
Warm neutrals with earthy accents. For lifestyle brands, portfolios, and organic products.
- Background: `#f5f0e8` | Surface: `#ede6d6` | Border: `#d4c9b0`
- Accent: `#8b5e3c` (terracotta) | Text: `#2c1810` | Muted: `#9a8878`
- Fonts: `Cormorant Garamond` (headings) + `Nunito` (body)
- Vibe: warm, handcrafted, grounded

### 5. Blueprint
Technical, structured, data-forward. For dashboards, admin panels, and SaaS tools.
- Background: `#f8fafc` | Surface: `#ffffff` | Border: `#e2e8f0`
- Accent: `#2563eb` (blue) | Text: `#0f172a` | Muted: `#64748b`
- Fonts: `IBM Plex Sans` (headings) + `IBM Plex Sans` (body)
- Vibe: clean, trustworthy, functional

### 6. Neon Noir
Dark cyberpunk aesthetic. For games, creative tools, and edgy brands.
- Background: `#0a0a0f` | Surface: `#12121a` | Border: `#1e1e2e`
- Accent: `#00fff5` (cyan) | Secondary: `#ff00aa` (magenta) | Text: `#e0e0ff`
- Fonts: `Orbitron` (headings) + `Rajdhani` (body)
- Vibe: futuristic, high-energy, dramatic

### 7. Parchment
Vintage, warm, nostalgic. For historical content, journals, and artisan brands.
- Background: `#f4ead5` | Surface: `#ede0c4` | Border: `#c9b99a`
- Accent: `#5c4a1e` (dark gold) | Text: `#2d1f0e` | Muted: `#8a7560`
- Fonts: `IM Fell English` (headings) + `Lora` (body)
- Vibe: aged, handwritten, timeless

### 8. Glacier
Cool, airy, Scandinavian. For wellness, productivity, and minimal SaaS.
- Background: `#f0f4f8` | Surface: `#ffffff` | Border: `#cdd8e3`
- Accent: `#0891b2` (teal) | Text: `#1e293b` | Muted: `#94a3b8`
- Fonts: `Outfit` (headings) + `Outfit` (body)
- Vibe: calm, spacious, focused

## Creating a Custom Theme

If none of the above fit, generate a custom theme based on the user's description:

1. Choose a name that captures the vibe
2. Define: background, surface, border, accent, text, muted
3. Pick a font pairing (display + body) from Google Fonts
4. Describe the vibe in one sentence
5. Show it to the user for approval before applying

## Application Rules

When applying a theme:
- Use CSS custom properties (`--color-bg`, `--color-accent`, etc.) for all values
- Load fonts via Google Fonts `@import` or `<link>`
- Apply consistently — no mixing of theme colors with hardcoded values
- Ensure contrast ratios: body text ≥ 4.5:1, large text ≥ 3:1
- Spacing scale: `4px` base unit, multiples of 4 throughout
