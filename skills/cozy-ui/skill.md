Design warm, soft, inviting interfaces with a cozy, hygge-inspired aesthetic. Use this skill when building personal apps, journaling tools, reading experiences, wellness products, home dashboards, or any interface that should feel like a warm cup of tea — comfortable, unhurried, and human.

## The Cozy Aesthetic

Cozy UI is the opposite of cold, corporate design. It prioritizes:
- **Warmth** — amber, cream, terracotta, sage, dusty rose over clinical blues and grays
- **Softness** — rounded corners, gentle shadows, no hard edges
- **Texture** — paper grain, linen, wood — surfaces that feel touchable
- **Imperfection** — hand-drawn elements, slightly irregular shapes, organic curves
- **Calm** — generous whitespace, unhurried pacing, no aggressive CTAs

## Color Palettes

### Warm Cream (default cozy)
```css
:root {
  --bg:        #faf6f0;
  --surface:   #f5ede0;
  --border:    #e8d8c4;
  --text:      #3d2b1f;
  --muted:     #9a8070;
  --accent:    #c17f4a;   /* warm amber */
  --accent-2:  #8faa7c;   /* sage green */
  --accent-3:  #c4847a;   /* dusty rose */
}
```

### Evening Hearth (darker, candlelit)
```css
:root {
  --bg:        #1e1510;
  --surface:   #2a1e16;
  --border:    #3d2e22;
  --text:      #f0e6d8;
  --muted:     #8a7060;
  --accent:    #e8a050;
  --accent-2:  #7a9e6a;
}
```

### Misty Morning (cool-warm blend)
```css
:root {
  --bg:        #f4f0ec;
  --surface:   #ece6de;
  --border:    #d8cfc4;
  --text:      #2e2820;
  --muted:     #8a8078;
  --accent:    #7a9e8a;   /* eucalyptus */
  --accent-2:  #b8956a;
}
```

## Typography

Cozy type feels handcrafted and personal:

```css
/* Heading: serif with character */
font-family: 'Lora', 'Playfair Display', 'Crimson Pro', Georgia, serif;

/* Body: warm, readable */
font-family: 'Source Serif 4', 'Merriweather', 'Libre Baskerville', serif;

/* Accent/labels: gentle sans */
font-family: 'Nunito', 'Quicksand', 'Jost', sans-serif;

/* Handwritten touches */
font-family: 'Caveat', 'Kalam', 'Patrick Hand', cursive;
```

Line height: 1.7–1.9 for body text. Cozy text breathes.

## Spacing & Shape

```css
/* Generous, unhurried spacing */
--space-xs:  6px;
--space-sm:  12px;
--space-md:  20px;
--space-lg:  36px;
--space-xl:  60px;

/* Soft, rounded corners */
--radius-sm:  8px;
--radius-md:  16px;
--radius-lg:  24px;
--radius-xl:  32px;
--radius-pill: 9999px;

/* Gentle shadows — warm-tinted, never cold gray */
--shadow-sm: 0 2px 8px rgba(100, 60, 20, 0.08);
--shadow-md: 0 4px 20px rgba(100, 60, 20, 0.12);
--shadow-lg: 0 8px 40px rgba(100, 60, 20, 0.16);
```

## Texture & Atmosphere

Add tactile depth with CSS textures:

```css
/* Paper grain overlay */
.paper-texture::before {
  content: '';
  position: absolute;
  inset: 0;
  background-image: url("data:image/svg+xml,%3Csvg viewBox='0 0 200 200' xmlns='http://www.w3.org/2000/svg'%3E%3Cfilter id='n'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.75' numOctaves='4' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23n)' opacity='0.4'/%3E%3C/svg%3E");
  opacity: 0.035;
  pointer-events: none;
  border-radius: inherit;
}

/* Warm vignette */
.cozy-container {
  background: radial-gradient(ellipse at center, transparent 60%, rgba(100, 60, 20, 0.06) 100%);
}

/* Subtle inner glow on cards */
.cozy-card {
  box-shadow:
    var(--shadow-md),
    inset 0 1px 0 rgba(255, 255, 255, 0.6);
}
```

## Component Patterns

### Cozy card
```css
.cozy-card {
  background: var(--surface);
  border: 1px solid var(--border);
  border-radius: var(--radius-lg);
  padding: var(--space-lg);
  box-shadow: var(--shadow-md);
  transition: transform 0.3s ease, box-shadow 0.3s ease;
}
.cozy-card:hover {
  transform: translateY(-2px);
  box-shadow: var(--shadow-lg);
}
```

### Soft button
```css
.btn-cozy {
  background: var(--accent);
  color: #fff;
  border: none;
  border-radius: var(--radius-pill);
  padding: 10px 24px;
  font-family: 'Nunito', sans-serif;
  font-weight: 600;
  letter-spacing: 0.02em;
  box-shadow: 0 2px 8px rgba(193, 127, 74, 0.3);
  transition: all 0.2s ease;
}
.btn-cozy:hover {
  transform: translateY(-1px);
  box-shadow: 0 4px 16px rgba(193, 127, 74, 0.4);
}
```

### Handwritten accent
```css
.handwritten-note {
  font-family: 'Caveat', cursive;
  font-size: 1.1rem;
  color: var(--accent);
  transform: rotate(-1.5deg);
  display: inline-block;
}
```

## Micro-interactions

Cozy interactions are gentle, never jarring:
- Hover: `translateY(-2px)` + slightly deeper shadow
- Click: `scale(0.97)` + `0.1s` ease
- Focus: warm-colored ring (`box-shadow: 0 0 0 3px rgba(193, 127, 74, 0.3)`)
- Loading: slow, gentle pulse (`opacity 1.5s ease-in-out infinite`)
- Transitions: `0.25–0.4s ease` — never instant, never slow

## Decorative Elements

Small touches that make an interface feel handcrafted:
- Dividers: wavy SVG lines instead of straight `<hr>`
- Bullet points: `✦`, `◦`, `·`, `❋` instead of default dots
- Empty states: illustrated, warm, encouraging ("Nothing here yet — enjoy the quiet")
- Icons: rounded, slightly chunky stroke weight (2px), warm color
- Badges: pill-shaped, pastel background, no harsh borders
