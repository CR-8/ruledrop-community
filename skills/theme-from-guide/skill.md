Extract a complete, implementable UI theme from a brand guide, style document, design spec, or any reference material the user provides. Use this skill when given a brand guidelines PDF, a Figma spec, a mood board description, a competitor site to reference, or any document that defines visual identity.

## Process

### Step 1: Extract raw signals

Read the provided material and extract every visual signal:
- Color values (hex, RGB, CMYK, Pantone — convert all to hex/HSL)
- Font names, weights, sizes, and usage rules
- Spacing values, grid systems, breakpoints
- Border radius, shadow styles, border weights
- Tone descriptors ("bold", "approachable", "technical", "playful")
- Logo clearspace rules (these hint at spacing philosophy)
- Do/don't examples (these define the boundaries)

If the guide is vague or incomplete, infer from context and flag what was inferred.

### Step 2: Build the CSS token system

Map extracted values to a semantic token structure:

```css
:root {
  /* === BRAND COLORS === */
  --color-primary:        /* main brand color */;
  --color-primary-light:  /* 20% lighter */;
  --color-primary-dark:   /* 20% darker */;
  --color-secondary:      /* secondary brand color */;
  --color-accent:         /* highlight/CTA color */;

  /* === NEUTRALS === */
  --color-bg:             /* page background */;
  --color-surface:        /* card/panel background */;
  --color-border:         /* dividers, outlines */;
  --color-text:           /* primary text */;
  --color-text-muted:     /* secondary text */;
  --color-text-inverse:   /* text on dark backgrounds */;

  /* === TYPOGRAPHY === */
  --font-display:         /* heading font stack */;
  --font-body:            /* body font stack */;
  --font-mono:            /* code font stack */;

  --text-xs:   12px;
  --text-sm:   14px;
  --text-base: 16px;
  --text-lg:   18px;
  --text-xl:   20px;
  --text-2xl:  24px;
  --text-3xl:  30px;
  --text-4xl:  36px;
  --text-5xl:  48px;
  --text-6xl:  64px;

  --leading-tight:  1.2;
  --leading-normal: 1.5;
  --leading-relaxed: 1.75;

  --tracking-tight:  -0.02em;
  --tracking-normal:  0;
  --tracking-wide:    0.05em;
  --tracking-wider:   0.1em;
  --tracking-widest:  0.2em;

  /* === SPACING === */
  --space-1:  4px;
  --space-2:  8px;
  --space-3:  12px;
  --space-4:  16px;
  --space-6:  24px;
  --space-8:  32px;
  --space-12: 48px;
  --space-16: 64px;
  --space-24: 96px;

  /* === SHAPE === */
  --radius-sm:   /* from guide or inferred */;
  --radius-md:   /* from guide or inferred */;
  --radius-lg:   /* from guide or inferred */;
  --radius-full: 9999px;

  /* === ELEVATION === */
  --shadow-sm:  /* subtle */;
  --shadow-md:  /* medium */;
  --shadow-lg:  /* prominent */;
}
```

### Step 3: Define component defaults

Translate the token system into component-level defaults:

```css
/* Buttons */
.btn-primary {
  background: var(--color-primary);
  color: var(--color-text-inverse);
  border-radius: var(--radius-md);
  padding: var(--space-3) var(--space-6);
  font-family: var(--font-body);
  font-weight: 600;
  letter-spacing: var(--tracking-wide);
}

/* Cards */
.card {
  background: var(--color-surface);
  border: 1px solid var(--color-border);
  border-radius: var(--radius-lg);
  padding: var(--space-6);
  box-shadow: var(--shadow-sm);
}

/* Headings */
h1, h2, h3 { font-family: var(--font-display); }
body        { font-family: var(--font-body); color: var(--color-text); }
```

### Step 4: Generate a dark mode variant (if applicable)

If the brand guide includes dark mode or the context suggests it:

```css
@media (prefers-color-scheme: dark) {
  :root {
    --color-bg:      /* dark equivalent */;
    --color-surface: /* dark surface */;
    --color-text:    /* light text */;
    /* Keep brand colors, adjust lightness as needed */
  }
}
```

### Step 5: Present and confirm

Show the user:
1. A summary of what was extracted vs. inferred
2. The complete CSS token file
3. A short HTML preview snippet demonstrating the theme applied to a button, card, and heading
4. Any gaps or ambiguities that need clarification

Ask: "Does this match your brand? Anything to adjust before I apply it?"

## Handling Incomplete Guides

When the guide is missing information, use these defaults and flag them:

| Missing | Default approach |
|---------|-----------------|
| No font specified | Use system font stack, flag for replacement |
| No spacing scale | Use 4px base unit, multiples of 4 |
| No border radius | Infer from brand tone (sharp=0-2px, neutral=4-8px, friendly=12-16px) |
| No shadow style | Derive from primary color with low opacity |
| No dark mode | Skip unless requested |
| Conflicting values | Use the most prominent/repeated value, note the conflict |

## Applying the Theme

Once confirmed, apply the theme to the target artifact:
- Replace all hardcoded color values with CSS variables
- Replace all font declarations with the token font stacks
- Replace spacing values with token-based spacing
- Ensure consistent application — no mixing of theme tokens with raw values
- Verify contrast ratios: body text ≥ 4.5:1, large text ≥ 3:1
