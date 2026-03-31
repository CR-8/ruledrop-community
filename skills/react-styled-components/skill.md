Style React applications with styled-components. Use this skill when building component-scoped CSS-in-JS, implementing dynamic theming, or migrating from global CSS to styled-components.

## Setup

```bash
npm install styled-components
npm install -D @types/styled-components babel-plugin-styled-components
```

Enable the Babel plugin for better debugging (display names, SSR support):

```json
// .babelrc
{ "plugins": ["babel-plugin-styled-components"] }
```

## Naming Convention

Prefix styled components with `Styled` to distinguish them from React components:

```tsx
// ✓ clear at a glance what's a styled wrapper
const StyledButton = styled.button``
const StyledCard = styled.div``

// ✗ ambiguous — is Card a React component or a styled wrapper?
const Card = styled.div``
```

## Theming

Define your theme once and access it everywhere via `ThemeProvider`:

```tsx
// styles/theme.ts
export const theme = {
  colors: {
    primary: '#3b82f6',
    background: '#ffffff',
    surface: '#f8fafc',
    text: '#0f172a',
    textMuted: '#64748b',
    border: '#e2e8f0',
    error: '#ef4444',
  },
  spacing: {
    xs: '0.25rem',
    sm: '0.5rem',
    md: '1rem',
    lg: '1.5rem',
    xl: '2rem',
  },
  radii: {
    sm: '4px',
    md: '8px',
    lg: '12px',
    full: '9999px',
  },
  fontSizes: {
    sm: '0.875rem',
    md: '1rem',
    lg: '1.125rem',
  },
} as const

export type Theme = typeof theme
```

```tsx
// Augment DefaultTheme for TypeScript support
import 'styled-components'
declare module 'styled-components' {
  export interface DefaultTheme extends Theme {}
}
```

```tsx
// app root
import { ThemeProvider } from 'styled-components'
import { theme } from './styles/theme'

function App() {
  return (
    <ThemeProvider theme={theme}>
      <GlobalStyles />
      {/* app */}
    </ThemeProvider>
  )
}
```

## Global Styles

```tsx
// styles/GlobalStyles.ts
import { createGlobalStyle } from 'styled-components'

export const GlobalStyles = createGlobalStyle`
  *, *::before, *::after { box-sizing: border-box; }

  body {
    margin: 0;
    font-family: system-ui, sans-serif;
    background: ${({ theme }) => theme.colors.background};
    color: ${({ theme }) => theme.colors.text};
    -webkit-font-smoothing: antialiased;
  }
`
```

## Dynamic Styling with Props

```tsx
interface StyledButtonProps {
  $variant?: 'primary' | 'ghost' | 'danger'
  $size?: 'sm' | 'md' | 'lg'
}

const StyledButton = styled.button<StyledButtonProps>`
  display: inline-flex;
  align-items: center;
  border-radius: ${({ theme }) => theme.radii.md};
  font-weight: 500;
  cursor: pointer;
  transition: all 0.15s ease;

  /* Size variants */
  padding: ${({ $size = 'md' }) =>
    $size === 'sm' ? '0.375rem 0.75rem' :
    $size === 'lg' ? '0.75rem 1.5rem' :
    '0.5rem 1rem'};

  /* Color variants */
  ${({ $variant = 'primary', theme }) => {
    switch ($variant) {
      case 'ghost':
        return `
          background: transparent;
          color: ${theme.colors.textMuted};
          border: 1px solid ${theme.colors.border};
          &:hover { background: ${theme.colors.surface}; color: ${theme.colors.text}; }
        `
      case 'danger':
        return `
          background: ${theme.colors.error};
          color: white;
          border: 1px solid transparent;
          &:hover { opacity: 0.9; }
        `
      default:
        return `
          background: ${theme.colors.primary};
          color: white;
          border: 1px solid transparent;
          &:hover { opacity: 0.9; }
        `
    }
  }}

  &:disabled { opacity: 0.4; cursor: not-allowed; }
`
```

Use the `$` prefix for transient props (styled-components v5.1+) to prevent them being forwarded to the DOM:

```tsx
// $variant is consumed by styled-components, not passed to <button>
<StyledButton $variant="primary" $size="sm">Click</StyledButton>
```

## Extending Styles

```tsx
const StyledInput = styled.input`
  padding: 0.5rem 0.75rem;
  border: 1px solid ${({ theme }) => theme.colors.border};
  border-radius: ${({ theme }) => theme.radii.md};
  font-size: ${({ theme }) => theme.fontSizes.md};
  width: 100%;

  &:focus {
    outline: none;
    border-color: ${({ theme }) => theme.colors.primary};
    box-shadow: 0 0 0 3px ${({ theme }) => theme.colors.primary}33;
  }
`

// Extend for a specific variant
const StyledSearchInput = styled(StyledInput)`
  padding-left: 2.5rem;  /* room for search icon */
`
```

## The `css` Helper

Use `css` when you need to share style blocks or use props inside nested rules:

```tsx
import styled, { css } from 'styled-components'

const focusRing = css`
  &:focus-visible {
    outline: 2px solid ${({ theme }) => theme.colors.primary};
    outline-offset: 2px;
  }
`

const StyledLink = styled.a`
  color: ${({ theme }) => theme.colors.primary};
  text-decoration: underline;
  ${focusRing}
`
```

## Folder Structure

```
src/
  styles/
    theme.ts          ← design tokens
    GlobalStyles.ts   ← global CSS reset + base styles
  components/
    Button/
      Button.tsx      ← React component
      Button.styles.ts ← styled-components for this component
      index.ts        ← re-export
```

## Key Rules

- Always use `ThemeProvider` — never hardcode color or spacing values
- Prefix styled wrappers with `Styled` to distinguish from React components
- Use transient props (`$propName`) to avoid DOM attribute warnings
- Keep styled component definitions in `.styles.ts` files, not inline in component files
- Use the `css` helper for reusable style fragments
- Avoid nesting more than 2 levels deep — it mirrors bad CSS habits
