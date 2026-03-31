Choose and implement the right state management approach for React apps. Use this skill when deciding between local state, context, Zustand, or Redux Toolkit, or when refactoring tangled state logic.

## Decision Framework

Pick the simplest tool that solves the problem. Escalate only when you hit real pain.

| State type | Tool |
|---|---|
| UI state (open/closed, active tab) | `useState` |
| Derived values | `useMemo` |
| Complex local state with transitions | `useReducer` |
| Shared state, low-frequency updates | React Context |
| Shared state, high-frequency updates | Zustand |
| Server state (fetching, caching, sync) | TanStack Query |
| Large app with complex async flows | Redux Toolkit |

**Server state is not client state.** Don't put API responses in Zustand or Redux — use TanStack Query. It handles caching, revalidation, loading, and error states for you.

## useState Patterns

Keep state as local as possible. Lift only when two siblings genuinely need the same value.

```tsx
// Derived state — don't store it, compute it
const [items, setItems] = useState<Item[]>([])
const completedCount = items.filter(i => i.done).length  // not useState

// Object state — spread to update
const [form, setForm] = useState({ name: '', email: '' })
const handleChange = (field: string, value: string) =>
  setForm(prev => ({ ...prev, [field]: value }))
```

## useReducer for Complex Local State

Use when state transitions have names and multiple fields change together.

```tsx
type State = { count: number; step: number; history: number[] }
type Action =
  | { type: 'increment' }
  | { type: 'decrement' }
  | { type: 'set_step'; payload: number }
  | { type: 'reset' }

function reducer(state: State, action: Action): State {
  switch (action.type) {
    case 'increment':
      return { ...state, count: state.count + state.step, history: [...state.history, state.count] }
    case 'decrement':
      return { ...state, count: state.count - state.step }
    case 'set_step':
      return { ...state, step: action.payload }
    case 'reset':
      return { count: 0, step: 1, history: [] }
  }
}
```

## Context — Right and Wrong

Context is not a state manager. It's a dependency injection mechanism. It re-renders every consumer on every value change.

**Good uses:** theme, locale, current user, feature flags — things that change rarely.

**Bad uses:** frequently-updated state like form values, filter state, real-time data.

```tsx
// Split contexts by update frequency
const UserContext = createContext<User | null>(null)
const ThemeContext = createContext<Theme>('light')

// Custom hook — always wrap context access
function useUser() {
  const ctx = useContext(UserContext)
  if (!ctx) throw new Error('useUser must be used within UserProvider')
  return ctx
}
```

## Zustand

Minimal, fast, no boilerplate. Best for shared client state that updates frequently.

```tsx
import { create } from 'zustand'
import { devtools, persist } from 'zustand/middleware'

interface CartStore {
  items: CartItem[]
  addItem: (item: CartItem) => void
  removeItem: (id: string) => void
  clear: () => void
  total: () => number
}

const useCartStore = create<CartStore>()(
  devtools(
    persist(
      (set, get) => ({
        items: [],
        addItem: (item) => set(state => ({ items: [...state.items, item] })),
        removeItem: (id) => set(state => ({ items: state.items.filter(i => i.id !== id) })),
        clear: () => set({ items: [] }),
        total: () => get().items.reduce((sum, i) => sum + i.price * i.quantity, 0),
      }),
      { name: 'cart-storage' }
    )
  )
)
```

**Select only what you need** to avoid unnecessary re-renders:

```tsx
// ✗ subscribes to entire store
const store = useCartStore()

// ✓ only re-renders when items.length changes
const itemCount = useCartStore(state => state.items.length)
```

## Redux Toolkit

Use for large apps with complex async flows, many developers, or when you need time-travel debugging.

```tsx
// features/posts/postsSlice.ts
import { createSlice, createAsyncThunk } from '@reduxjs/toolkit'

export const fetchPosts = createAsyncThunk('posts/fetchAll', async () => {
  const res = await fetch('/api/posts')
  return res.json()
})

const postsSlice = createSlice({
  name: 'posts',
  initialState: { items: [] as Post[], status: 'idle' as 'idle' | 'loading' | 'succeeded' | 'failed' },
  reducers: {
    postAdded: (state, action) => { state.items.push(action.payload) },
  },
  extraReducers: (builder) => {
    builder
      .addCase(fetchPosts.pending, (state) => { state.status = 'loading' })
      .addCase(fetchPosts.fulfilled, (state, action) => {
        state.status = 'succeeded'
        state.items = action.payload
      })
  },
})
```

Use typed hooks — never use raw `useDispatch`/`useSelector`:

```tsx
// store/hooks.ts
import { useDispatch, useSelector } from 'react-redux'
import type { RootState, AppDispatch } from './store'

export const useAppDispatch = () => useDispatch<AppDispatch>()
export const useAppSelector = <T>(selector: (state: RootState) => T) => useSelector(selector)
```

## TanStack Query for Server State

```tsx
// Fetching
const { data, isLoading, error } = useQuery({
  queryKey: ['posts', filters],
  queryFn: () => fetchPosts(filters),
  staleTime: 1000 * 60 * 5, // 5 minutes
})

// Mutations with optimistic updates
const mutation = useMutation({
  mutationFn: createPost,
  onSuccess: () => queryClient.invalidateQueries({ queryKey: ['posts'] }),
})
```

## Key Rules

- Co-locate state with the component that owns it — don't hoist prematurely
- Never store derived values in state — compute them
- Don't put server responses in Zustand/Redux — that's TanStack Query's job
- Zustand slices should be small and focused — one concern per store
- Always use typed selectors in Redux — `useAppSelector`, never raw `useSelector`
