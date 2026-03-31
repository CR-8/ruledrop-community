Apply Airbnb's engineering standards for JavaScript, React, and CSS. Use this skill when writing JavaScript or TypeScript that should follow Airbnb's widely-adopted style guide, building React applications to Airbnb's component standards, or setting up ESLint with Airbnb's config.

## Core Philosophy

Airbnb's style guide is the most widely adopted JavaScript style guide in the world. Its principles:
- **Explicit over implicit** — code should be obvious to a new reader
- **Consistency at scale** — rules exist to make a large codebase feel like one person wrote it
- **Modern JS** — embrace ES6+ features that improve clarity; avoid those that obscure it
- **Accessibility first** — a11y is not optional

## JavaScript Style

### Variables
```js
// Always const first, let when reassignment is needed, never var
const MAX_SIZE = 100
let count = 0

// Destructure when accessing multiple properties
const { name, email, role } = user
const [first, ...rest] = items

// Use default parameters instead of conditional assignment
function createUser(name, role = 'viewer') { ... }

// Prefer template literals over concatenation
const greeting = `Hello, ${name}!`
```

### Functions
```js
// Named function expressions over anonymous
// BAD:
const foo = function() {}
// GOOD:
const foo = function namedFoo() {}

// Arrow functions for callbacks and short functions
const doubled = numbers.map(n => n * 2)
const isEven = n => n % 2 === 0

// Avoid arrow functions for methods that use `this`
const obj = {
  value: 42,
  getValue() { return this.value },  // GOOD
  // getValue: () => this.value,     // BAD — `this` is wrong
}

// IIFE: use arrow function form
(() => { ... })()
```

### Objects & Arrays
```js
// Shorthand properties
const name = 'Alice'
const user = { name, age: 30 }  // not { name: name, age: 30 }

// Computed property names
const key = 'email'
const obj = { [key]: 'alice@example.com' }

// Spread over Object.assign
const merged = { ...defaults, ...overrides }

// Array spread over slice/concat
const copy = [...original]
const combined = [...arr1, ...arr2]

// Use Array methods over loops
const names = users.map(u => u.name)
const admins = users.filter(u => u.role === 'admin')
const total = orders.reduce((sum, o) => sum + o.amount, 0)
```

### Classes
```js
// Use class syntax, not prototype manipulation
class Animal {
  #name  // private field

  constructor(name) {
    this.#name = name
  }

  speak() {
    return `${this.#name} makes a sound`
  }
}

class Dog extends Animal {
  speak() {
    return `${super.speak()} — woof!`
  }
}

// Avoid unnecessary constructors
// BAD: constructor that just calls super
class MyComponent extends Component {
  constructor(props) {
    super(props)  // unnecessary if no other logic
  }
}
```

### Modules
```js
// Named exports over default exports (easier to refactor, better IDE support)
// PREFER:
export function calculateTotal(items) { ... }
export const TAX_RATE = 0.08

// AVOID (when possible):
export default function() { ... }

// Import what you use
import { useState, useEffect } from 'react'  // not import React from 'react'

// Group imports: external → internal → relative
import React from 'react'
import { motion } from 'framer-motion'

import { UserService } from '@/services/user'
import { formatDate } from '@/utils/date'

import { Avatar } from './Avatar'
import styles from './UserCard.module.css'
```

## React Style (Airbnb React/JSX Guide)

### Component structure
```tsx
// File: one component per file, filename matches component name
// UserCard.tsx → export function UserCard

// Props: destructure in signature, define interface above component
interface UserCardProps {
  user: User
  onSelect?: (user: User) => void
  className?: string
}

function UserCard({ user, onSelect, className }: UserCardProps) {
  // 1. Hooks
  const [isExpanded, setIsExpanded] = useState(false)

  // 2. Derived state / memos
  const displayName = user.firstName + ' ' + user.lastName

  // 3. Handlers
  const handleClick = () => {
    setIsExpanded(prev => !prev)
    onSelect?.(user)
  }

  // 4. Render
  return (
    <div className={cn('user-card', className)} onClick={handleClick}>
      <Avatar src={user.avatarUrl} alt={displayName} />
      <span>{displayName}</span>
    </div>
  )
}
```

### JSX rules
```tsx
// Self-close tags with no children
<Component />  // not <Component></Component>

// Multiline JSX: wrap in parens, one prop per line when > 2 props
return (
  <UserCard
    user={currentUser}
    onSelect={handleSelect}
    className="featured"
  />
)

// Keys: use stable IDs, never array index (unless list is static and never reordered)
{users.map(user => <UserCard key={user.id} user={user} />)}

// Avoid inline functions in JSX when they cause unnecessary re-renders
// BAD:  onClick={() => handleClick(item.id)}
// GOOD: define handler outside JSX or use useCallback
```

### Hooks rules
```tsx
// Only call hooks at the top level — never inside conditions or loops
// Only call hooks from React functions — not regular JS functions

// Custom hooks: always prefix with 'use'
function useWindowSize() {
  const [size, setSize] = useState({ width: 0, height: 0 })
  useEffect(() => {
    const update = () => setSize({ width: window.innerWidth, height: window.innerHeight })
    window.addEventListener('resize', update)
    update()
    return () => window.removeEventListener('resize', update)
  }, [])
  return size
}
```

## CSS / Styling

Airbnb uses CSS Modules + PostCSS internally:

```css
/* BEM-inspired naming within modules */
.card { ... }
.card__header { ... }
.card__body { ... }
.card--featured { ... }

/* Use CSS custom properties for theming */
.card {
  background: var(--color-surface);
  border-radius: var(--radius-md);
  padding: var(--space-4);
}

/* Avoid magic numbers — use tokens */
/* BAD:  margin-top: 24px; */
/* GOOD: margin-top: var(--space-6); */
```

## ESLint Config

```bash
npm install --save-dev eslint-config-airbnb eslint-config-airbnb-typescript
```

```json
{
  "extends": [
    "airbnb",
    "airbnb-typescript",
    "airbnb/hooks"
  ],
  "rules": {
    "react/react-in-jsx-scope": "off",
    "import/prefer-default-export": "off"
  }
}
```

## Accessibility (Airbnb's a11y commitment)

```tsx
// Every interactive element needs accessible text
<button aria-label="Close dialog">×</button>

// Images need alt text
<img src={user.avatar} alt={`${user.name}'s profile photo`} />

// Form inputs need labels
<label htmlFor="email">Email</label>
<input id="email" type="email" />

// Use semantic HTML
<nav>, <main>, <article>, <section>, <aside>  // not just <div>

// Keyboard navigation: all interactive elements must be reachable via Tab
// Focus indicators must be visible (never outline: none without replacement)
```
