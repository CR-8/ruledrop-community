Build scroll-driven, narrative web experiences where the page itself tells a story. Use this skill when creating editorial websites, interactive longforms, product launch pages, portfolio showcases, or any site where the user's journey through the page is the experience.

## Core Concept

Immersive storytelling websites treat scroll as a timeline. Every pixel scrolled advances the narrative. The goal is to make the user feel like they're moving through a world, not reading a document.

Key principles:
- **Scroll = time** — content reveals, transforms, and transitions as the user scrolls
- **Sections feel like scenes** — each section has a distinct mood, not just different content
- **Motion has meaning** — every animation communicates something, nothing moves for decoration alone
- **Performance is non-negotiable** — jank destroys immersion instantly

## Scroll Animation Stack

### GSAP ScrollTrigger (recommended)

```js
import gsap from 'gsap'
import ScrollTrigger from 'gsap/ScrollTrigger'
gsap.registerPlugin(ScrollTrigger)

// Pin a section and animate within it
gsap.timeline({
  scrollTrigger: {
    trigger: '#scene-1',
    start: 'top top',
    end: '+=200%',
    pin: true,
    scrub: 1,
  }
})
.from('.headline', { y: 60, opacity: 0, duration: 0.3 })
.from('.subtext', { y: 40, opacity: 0, duration: 0.3 }, '-=0.1')
.to('.background', { scale: 1.15, duration: 1 }, 0)

// Horizontal scroll section
gsap.to('.panels', {
  xPercent: -100 * (panelCount - 1),
  ease: 'none',
  scrollTrigger: {
    trigger: '.panels-container',
    pin: true,
    scrub: 1,
    snap: 1 / (panelCount - 1),
    end: () => '+=' + document.querySelector('.panels-container').offsetWidth
  }
})
```

### Intersection Observer (lightweight alternative)

```js
const observer = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      entry.target.classList.add('in-view')
    }
  })
}, { threshold: 0.2, rootMargin: '0px 0px -10% 0px' })

document.querySelectorAll('[data-reveal]').forEach(el => observer.observe(el))
```

```css
[data-reveal] {
  opacity: 0;
  transform: translateY(40px);
  transition: opacity 0.8s ease, transform 0.8s ease;
}
[data-reveal].in-view {
  opacity: 1;
  transform: none;
}
```

## Scene Patterns

### Parallax depth layers

```css
.scene {
  position: relative;
  height: 100vh;
  overflow: hidden;
}
.layer-back   { transform: translateY(calc(var(--scroll) * 0.3px)); }
.layer-mid    { transform: translateY(calc(var(--scroll) * 0.6px)); }
.layer-front  { transform: translateY(calc(var(--scroll) * 1.0px)); }
```

```js
// Update CSS variable on scroll (use requestAnimationFrame)
let ticking = false
window.addEventListener('scroll', () => {
  if (!ticking) {
    requestAnimationFrame(() => {
      document.documentElement.style.setProperty('--scroll', window.scrollY)
      ticking = false
    })
    ticking = true
  }
})
```

### Text reveal — word by word

```js
function splitAndReveal(el) {
  const words = el.textContent.split(' ')
  el.innerHTML = words.map(w => `<span class="word"><span>${w}</span></span>`).join(' ')

  gsap.from(el.querySelectorAll('.word span'), {
    y: '100%',
    opacity: 0,
    stagger: 0.04,
    duration: 0.6,
    ease: 'power3.out',
    scrollTrigger: { trigger: el, start: 'top 80%' }
  })
}
```

### Sticky narrative — content changes while hero stays pinned

```html
<section class="sticky-container">
  <div class="sticky-visual"><!-- 3D scene or video --></div>
  <div class="scroll-steps">
    <div class="step" data-step="1">First revelation</div>
    <div class="step" data-step="2">Second revelation</div>
    <div class="step" data-step="3">Third revelation</div>
  </div>
</section>
```

```js
// Swap visual state as each step enters view
ScrollTrigger.create({
  trigger: '.scroll-steps',
  start: 'top top',
  end: 'bottom bottom',
  onUpdate: (self) => {
    const step = Math.floor(self.progress * 3)
    updateVisual(step)
  }
})
```

### Full-screen video/canvas transitions

```css
.scene-transition {
  clip-path: circle(0% at 50% 50%);
  transition: clip-path 1.2s cubic-bezier(0.77, 0, 0.175, 1);
}
.scene-transition.active {
  clip-path: circle(150% at 50% 50%);
}
```

## Typography for Storytelling

- **Display type is the hero** — use large, expressive type (100px+) as a visual element, not just text
- **Variable fonts** — animate `font-weight`, `font-stretch` on scroll for kinetic typography
- **Contrast rhythm** — alternate between dense text sections and open, image-heavy sections
- **Reading pace** — short lines (45–65 chars) for narrative text; wide lines for impact statements

```css
/* Kinetic variable font on scroll */
.kinetic-headline {
  font-variation-settings: 'wght' var(--font-weight, 100);
  transition: font-variation-settings 0.3s ease;
}
```

## Atmosphere & Mood

Each scene should have a distinct atmosphere:

- **Color temperature shifts** — warm → cool → warm as narrative progresses
- **Ambient sound** (optional, always muted by default, user-initiated)
- **Cursor customization** — custom cursor that reacts to scene context
- **Grain & texture overlays** — add film grain to create depth and warmth
- **Vignette** — subtle dark edges focus attention on center content

```css
/* Grain overlay */
.grain::after {
  content: '';
  position: fixed;
  inset: 0;
  background-image: url("data:image/svg+xml,..."); /* noise SVG */
  opacity: 0.04;
  pointer-events: none;
  z-index: 9999;
}
```

## Performance for Immersive Sites

- **Preload critical assets** — fonts, hero images, first-scene video
- **Lazy load below-fold scenes** — use `loading="lazy"` and IntersectionObserver
- **Avoid layout thrash** — batch DOM reads/writes, never read then write in a loop
- **Use `will-change` sparingly** — only on elements actively animating, remove after
- **Reduce motion** — always respect `prefers-reduced-motion`

```css
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    transition-duration: 0.01ms !important;
  }
}
```

## Structure Template

```
/
├── Scene 1: Hook — full-screen, immediate impact, 3–5 words
├── Scene 2: Context — who/what/why, parallax depth
├── Scene 3: Tension — the problem or journey
├── Scene 4: Resolution — the product/idea/moment
├── Scene 5: Call to action — simple, uncluttered
```

Each scene: one idea, one emotion, one visual focus.
