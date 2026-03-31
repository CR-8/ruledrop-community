Apply proven React and TypeScript component patterns for maintainable, performant frontends. Use this skill when building React components, managing state, fetching data, optimizing performance, handling forms, or structuring a frontend codebase.

## Component Patterns

### Composition over inheritance

Build UIs by composing small, focused components. Avoid deep inheritance chains.

```tsx
// Slot-based composition
function Card({ children }: { children: React.ReactNode }) {
  return <div className="card">{children}</div>
}
function CardHeader({ children }: { children: React.ReactNode }) {
  return <div className="card-header">{children}</div>
}
function CardBody({ children }: { children: React.ReactNode }) {
  return <div className="card-body">{children}</div>
}
```

### Compound components

Share implicit state between related components via context.

```tsx
const TabsContext = createContext<{ active: string; setActive: (id: string) => void } | null>(null)

function Tabs({ children, defaultTab }: { children: React.ReactNode; defaultTab: string }) {
  const [active, setActive] = useState(defaultTab)
  return <TabsContext.Provider value={{ active, setActive }}>{children}</TabsContext.Provider>
}

function Tab({ id, children }: { id: string; children: React.ReactNode }) {
  const ctx = useContext(TabsContext)!
  return (
    <button className={ctx.active === id ? 'active' : ''} onClick={() => ctx.setActive(id)}>
      {children}
    </button>
  )
}
```

## Custom Hook Patterns

### Async data fetching

```tsx
function useQuery<T>(key: string, fetcher: () => Promise<T>) {
  const [data, setData] = useState<T | null>(null)
  const [loading, setLoading] = useState(true)
  const [error, setError] = useState<Error | null>(null)

  useEffect(() => {
    setLoading(true)
    fetcher()
      .then(setData)
      .catch(setError)
      .finally(() => setLoading(false))
  }, [key])

  return { data, loading, error }
}
```

### Debounce

```tsx
function useDebounce<T>(value: T, delay: number): T {
  const [debounced, setDebounced] = useState(value)
  useEffect(() => {
    const t = setTimeout(() => setDebounced(value), delay)
    return () => clearTimeout(t)
  }, [value, delay])
  return debounced
}
```

## State Management

### Context + Reducer for shared state

```tsx
type Action = { type: 'SET_ITEMS'; payload: Item[] } | { type: 'SELECT'; payload: Item }

function reducer(state: State, action: Action): State {
  switch (action.type) {
    case 'SET_ITEMS': return { ...state, items: action.payload }
    case 'SELECT':    return { ...state, selected: action.payload }
    default:          return state
  }
}
```

Use Zustand for larger apps where context prop-drilling becomes painful.

## Performance

### Memoization rules

- `useMemo` — expensive derivations (sorting, filtering large arrays)
- `useCallback` — functions passed as props to memoized children
- `React.memo` — pure components that receive stable props

Don't memoize everything. Profile first, optimize second.

### Code splitting

```tsx
const HeavyChart = lazy(() => import('./HeavyChart'))

function Dashboard() {
  return (
    <Suspense fallback={<Skeleton />}>
      <HeavyChart />
    </Suspense>
  )
}
```

### Virtualize long lists

Use `@tanstack/react-virtual` for lists over ~100 items. Never render thousands of DOM nodes.

## Form Handling

### Controlled form with validation

```tsx
function validate(data: FormData): Record<string, string> {
  const errors: Record<string, string> = {}
  if (!data.name.trim()) errors.name = 'Required'
  if (data.name.length > 200) errors.name = 'Max 200 characters'
  return errors
}
```

Use Zod + React Hook Form for complex forms. Keep validation logic outside components.

## Error Boundaries

Wrap feature sections in error boundaries so one failure doesn't crash the whole app.

```tsx
class ErrorBoundary extends React.Component<
  { children: React.ReactNode },
  { error: Error | null }
> {
  state = { error: null }
  static getDerivedStateFromError(error: Error) { return { error } }
  render() {
    if (this.state.error) return <ErrorFallback error={this.state.error} />
    return this.props.children
  }
}
```

## Accessibility

- Use semantic HTML (`button`, `nav`, `main`, `dialog`) before reaching for `div`
- Every interactive element needs keyboard support (`onKeyDown` for arrow keys, Enter, Escape)
- Modals must trap focus and restore it on close
- Use `aria-label`, `aria-expanded`, `aria-haspopup` on custom widgets
- Test with a keyboard before shipping

## Animation

Use Framer Motion for React. Prefer `AnimatePresence` for enter/exit transitions. One well-timed entrance animation beats five scattered micro-interactions.

```tsx
<AnimatePresence>
  {isOpen && (
    <motion.div
      initial={{ opacity: 0, y: 8 }}
      animate={{ opacity: 1, y: 0 }}
      exit={{ opacity: 0, y: 8 }}
    />
  )}
</AnimatePresence>
```
