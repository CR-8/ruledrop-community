Create expressive vector graphics, animated SVGs, and motion-driven UI using SVG, CSS, and animation libraries. Use this skill when building icon systems, illustrated UIs, animated logos, data visualizations, decorative backgrounds, or any interface where vector art and motion graphics play a central role.

## SVG Fundamentals

Write SVGs that are clean, accessible, and animatable:

```svg
<svg viewBox="0 0 100 100" xmlns="http://www.w3.org/2000/svg" role="img" aria-label="Description">
  <!-- Use semantic grouping -->
  <g id="background">
    <rect width="100" height="100" fill="url(#grad)" />
  </g>
  <g id="content">
    <circle cx="50" cy="50" r="30" fill="#6c63ff" />
  </g>

  <!-- Define reusables -->
  <defs>
    <linearGradient id="grad" x1="0%" y1="0%" x2="100%" y2="100%">
      <stop offset="0%" stop-color="#6c63ff" />
      <stop offset="100%" stop-color="#ff6584" />
    </linearGradient>
    <filter id="glow">
      <feGaussianBlur stdDeviation="3" result="blur" />
      <feMerge><feMergeNode in="blur" /><feMergeNode in="SourceGraphic" /></feMerge>
    </filter>
  </defs>
</svg>
```

## CSS SVG Animation

Animate SVG properties directly with CSS for lightweight, GPU-accelerated motion:

```css
/* Path drawing effect */
.line {
  stroke-dasharray: 300;
  stroke-dashoffset: 300;
  animation: draw 1.5s ease forwards;
}
@keyframes draw {
  to { stroke-dashoffset: 0; }
}

/* Morphing shapes */
@keyframes morph {
  0%   { d: path("M 10,30 A 20,20,0,0,1,50,30 A 20,20,0,0,1,90,30 Q 90,60,50,90 Q 10,60,10,30 Z"); }
  50%  { d: path("M 10,50 A 20,20,0,0,1,50,10 A 20,20,0,0,1,90,50 Q 90,80,50,90 Q 10,80,10,50 Z"); }
  100% { d: path("M 10,30 A 20,20,0,0,1,50,30 A 20,20,0,0,1,90,30 Q 90,60,50,90 Q 10,60,10,30 Z"); }
}
```

## GSAP for Complex SVG Animation

Use GSAP when you need sequenced, timeline-based, or scroll-triggered vector animation:

```js
import gsap from 'gsap'
import { DrawSVGPlugin, MorphSVGPlugin } from 'gsap/all'
gsap.registerPlugin(DrawSVGPlugin, MorphSVGPlugin)

// Staggered path reveal
gsap.from('.path', {
  drawSVG: '0%',
  duration: 1.5,
  stagger: 0.1,
  ease: 'power2.inOut'
})

// Shape morphing
gsap.to('#shape', {
  morphSVG: '#target-shape',
  duration: 1,
  ease: 'elastic.out(1, 0.5)'
})

// Timeline sequence
const tl = gsap.timeline({ defaults: { ease: 'power3.out' } })
tl.from('#logo', { scale: 0, opacity: 0, duration: 0.6 })
  .from('#tagline', { y: 20, opacity: 0, duration: 0.4 }, '-=0.2')
  .from('.nav-item', { y: -20, opacity: 0, stagger: 0.08 }, '-=0.2')
```

## Framer Motion SVG (React)

```jsx
import { motion } from 'framer-motion'

// Animated path
<motion.path
  d="M 10 80 Q 95 10 180 80"
  stroke="#6c63ff"
  strokeWidth="3"
  fill="none"
  initial={{ pathLength: 0 }}
  animate={{ pathLength: 1 }}
  transition={{ duration: 1.5, ease: 'easeInOut' }}
/>

// SVG icon with hover
<motion.svg
  whileHover={{ scale: 1.1, rotate: 5 }}
  whileTap={{ scale: 0.95 }}
  transition={{ type: 'spring', stiffness: 400 }}
>
  <path d="..." />
</motion.svg>
```

## Generative SVG Patterns

Create dynamic, code-generated backgrounds and textures:

```js
// Organic blob using cubic bezier curves
function generateBlob(points = 6, radius = 100, variance = 30) {
  const angleStep = (Math.PI * 2) / points
  const pts = Array.from({ length: points }, (_, i) => {
    const angle = i * angleStep
    const r = radius + (Math.random() - 0.5) * variance * 2
    return [Math.cos(angle) * r + 150, Math.sin(angle) * r + 150]
  })
  // Build smooth cubic bezier path through points
  return pts.map((p, i) => {
    const prev = pts[(i - 1 + points) % points]
    const next = pts[(i + 1) % points]
    const cp1 = [p[0] + (next[0] - prev[0]) * 0.2, p[1] + (next[1] - prev[1]) * 0.2]
    return i === 0 ? `M ${p[0]} ${p[1]}` : `C ${cp1[0]} ${cp1[1]}, ${p[0]} ${p[1]}, ${p[0]} ${p[1]}`
  }).join(' ') + ' Z'
}

// Noise texture overlay
// Use feaTurbulence SVG filter for organic grain
const noiseFilter = `
  <filter id="noise">
    <feTurbulence type="fractalNoise" baseFrequency="0.65" numOctaves="3" stitchTiles="stitch"/>
    <feColorMatrix type="saturate" values="0"/>
    <feBlend in="SourceGraphic" mode="multiply"/>
  </filter>
`
```

## Icon System Design

Build a consistent, scalable icon system:

```jsx
// Base icon component
function Icon({ name, size = 24, color = 'currentColor', ...props }) {
  return (
    <svg
      width={size}
      height={size}
      viewBox="0 0 24 24"
      fill="none"
      stroke={color}
      strokeWidth={1.5}
      strokeLinecap="round"
      strokeLinejoin="round"
      aria-hidden="true"
      {...props}
    >
      <use href={`/icons.svg#${name}`} />
    </svg>
  )
}
```

Rules for icon design:
- 24×24 viewBox, 1.5px stroke weight at default size
- Optical sizing — icons should feel the same visual weight at all sizes
- Consistent corner radius across the set
- Pixel-hinted at 16px and 24px sizes
- Export as SVG sprite for production (single HTTP request)

## Data Visualization with SVG

For charts without a library:

```jsx
// Simple bar chart
function BarChart({ data, width = 400, height = 200 }) {
  const max = Math.max(...data.map(d => d.value))
  const barWidth = width / data.length - 4

  return (
    <svg viewBox={`0 0 ${width} ${height}`}>
      {data.map((d, i) => {
        const barHeight = (d.value / max) * (height - 20)
        return (
          <g key={d.label} transform={`translate(${i * (barWidth + 4)}, 0)`}>
            <motion.rect
              x={0} y={height - barHeight - 20}
              width={barWidth} height={barHeight}
              fill="url(#barGrad)" rx={3}
              initial={{ scaleY: 0, originY: 1 }}
              animate={{ scaleY: 1 }}
              transition={{ delay: i * 0.05, duration: 0.5 }}
            />
            <text x={barWidth / 2} y={height - 4} textAnchor="middle" fontSize={10}>
              {d.label}
            </text>
          </g>
        )
      })}
    </svg>
  )
}
```

## Performance

- Inline critical SVGs; use `<img>` or CSS `background` for decorative ones
- Use `will-change: transform` on animated SVG elements
- Prefer CSS animations over JS for simple loops (GPU composited)
- Simplify paths — reduce node count with tools like SVGO before shipping
- Avoid animating `d` attribute directly in CSS (use GSAP MorphSVG or clip-path instead)
