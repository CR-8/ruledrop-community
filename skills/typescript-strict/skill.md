Write strict, idiomatic TypeScript. Use this skill when typing React components, designing interfaces, handling unknown data, narrowing types, or eliminating `any` from a codebase.

## tsconfig Baseline

Always enable strict mode. These options catch the most real bugs:

```json
{
  "compilerOptions": {
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "noImplicitReturns": true,
    "exactOptionalPropertyTypes": true,
    "moduleResolution": "bundler",
    "target": "ES2022"
  }
}
```

## Interfaces vs Types

- `interface` — for object shapes that may be extended or implemented
- `type` — for unions, intersections, mapped types, and aliases

```ts
// interface for extendable shapes
interface User {
  id: string
  name: string
  email: string
}

interface AdminUser extends User {
  permissions: string[]
}

// type for unions and computed shapes
type Status = 'idle' | 'loading' | 'success' | 'error'
type Nullable<T> = T | null
type PartialBy<T, K extends keyof T> = Omit<T, K> & Partial<Pick<T, K>>
```

## Eliminate `any`

`any` disables the type checker. Replace it:

| Instead of | Use |
|---|---|
| `any` for unknown input | `unknown` + type guard |
| `any[]` for mixed arrays | `unknown[]` or a union |
| `any` for JSON | `unknown` + Zod parse |
| `(e: any)` in catch | `(e: unknown)` + `instanceof Error` |

```ts
// ✗
function parse(data: any) { return data.name }

// ✓
function parse(data: unknown): string {
  if (typeof data !== 'object' || data === null) throw new Error('Invalid data')
  if (!('name' in data) || typeof (data as Record<string, unknown>).name !== 'string') {
    throw new Error('Missing name')
  }
  return (data as { name: string }).name
}
```

## Type Narrowing

Use narrowing instead of casting. TypeScript understands these patterns:

```ts
// typeof
function format(val: string | number) {
  if (typeof val === 'string') return val.toUpperCase()
  return val.toFixed(2)
}

// instanceof
function handleError(e: unknown) {
  if (e instanceof Error) return e.message
  return String(e)
}

// Discriminated union — the most powerful pattern
type Result<T> =
  | { status: 'success'; data: T }
  | { status: 'error'; error: string }
  | { status: 'loading' }

function render(result: Result<User>) {
  switch (result.status) {
    case 'success': return result.data.name  // data is typed here
    case 'error':   return result.error      // error is typed here
    case 'loading': return 'Loading...'
  }
}
```

## Generics

Write generic functions when the logic is the same but the type varies:

```ts
// Generic fetch wrapper
async function fetchJson<T>(url: string): Promise<T> {
  const res = await fetch(url)
  if (!res.ok) throw new Error(`HTTP ${res.status}`)
  return res.json() as Promise<T>
}

// Generic hook
function useLocalStorage<T>(key: string, initial: T) {
  const [value, setValue] = useState<T>(() => {
    try {
      const stored = localStorage.getItem(key)
      return stored ? (JSON.parse(stored) as T) : initial
    } catch {
      return initial
    }
  })
  // ...
}
```

Constrain generics when you need to access properties:

```ts
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key]
}
```

## React Component Typing

```tsx
// Props interface — always explicit
interface ButtonProps {
  label: string
  variant?: 'primary' | 'ghost' | 'danger'
  disabled?: boolean
  onClick?: () => void
  children?: React.ReactNode
}

// Extend native element props
interface InputProps extends React.InputHTMLAttributes<HTMLInputElement> {
  label: string
  error?: string
}

// Generic component
interface ListProps<T> {
  items: T[]
  renderItem: (item: T, index: number) => React.ReactNode
  keyExtractor: (item: T) => string
}

function List<T>({ items, renderItem, keyExtractor }: ListProps<T>) {
  return <ul>{items.map((item, i) => <li key={keyExtractor(item)}>{renderItem(item, i)}</li>)}</ul>
}
```

## Utility Types

Know the built-ins before writing your own:

```ts
Partial<T>           // all props optional
Required<T>          // all props required
Readonly<T>          // all props readonly
Pick<T, K>           // keep only K keys
Omit<T, K>           // remove K keys
Record<K, V>         // object with keys K and values V
Exclude<T, U>        // remove U from union T
Extract<T, U>        // keep only U from union T
NonNullable<T>       // remove null and undefined
ReturnType<F>        // return type of function F
Parameters<F>        // parameter types of function F
Awaited<T>           // unwrap Promise<T>
```

## Zod for Runtime Validation

TypeScript types disappear at runtime. Use Zod to validate external data:

```ts
import { z } from 'zod'

const UserSchema = z.object({
  id: z.string().uuid(),
  name: z.string().min(1).max(200),
  email: z.string().email(),
  role: z.enum(['user', 'admin']),
  createdAt: z.string().datetime(),
})

type User = z.infer<typeof UserSchema>  // derive type from schema

// Validate API response
const user = UserSchema.parse(await res.json())  // throws on invalid
const result = UserSchema.safeParse(data)         // returns { success, data, error }
```

## Key Rules

- `strict: true` is non-negotiable — enable it from day one
- Never use `as` to cast unless you've already narrowed the type
- Prefer discriminated unions over optional fields for state modeling
- Derive types from schemas (`z.infer`) — don't duplicate definitions
- Use `satisfies` operator to validate literals without widening the type
- Avoid `enum` — use `as const` objects or string literal unions instead
