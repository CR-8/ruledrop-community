Apply Meta's engineering standards for code quality, performance, and developer experience. Use this skill when writing code that should follow Meta's engineering culture, building React/React Native applications (Meta's primary frontend stack), or applying Meta's approach to scale, reliability, and developer tooling.

## Core Philosophy

Meta's engineering culture is shaped by its scale (billions of users) and speed (move fast):
- **Move fast with stable infrastructure** — velocity matters, but not at the cost of reliability
- **Data-driven decisions** — instrument everything, A/B test before shipping
- **Developer experience is a product** — internal tooling is treated as seriously as user-facing products
- **Graceful degradation** — systems must handle partial failures without full outages

## React Patterns (Meta's home stack)

Meta created React and its conventions reflect how they use it internally.

### Component design
```tsx
// Prefer function components with hooks — no class components
// Co-locate state with the component that owns it
// Lift state only when necessary

// GOOD: focused, single-responsibility component
function UserAvatar({ userId }: { userId: string }) {
  const user = useUser(userId)  // custom hook encapsulates data fetching
  if (!user) return <AvatarSkeleton />
  return <img src={user.avatarUrl} alt={user.name} className="avatar" />
}

// AVOID: components that do too much
function UserProfilePage() {
  // fetching, transforming, rendering, handling events — all in one component
}
```

### Relay-style data fetching (Meta's GraphQL client)
Even without Relay, apply its colocation principle:

```tsx
// Each component declares its own data requirements
// Parent composes fragments, doesn't pass down individual props

// Component declares what it needs
const USER_FRAGMENT = gql`
  fragment UserCard_user on User {
    id
    name
    avatarUrl
    bio
  }
`

function UserCard({ userRef }: { userRef: UserCard_user$key }) {
  const user = useFragment(USER_FRAGMENT, userRef)
  return <div>{user.name}</div>
}
```

### Performance — Meta's scale demands it
```tsx
// Virtualize any list over ~50 items
import { FixedSizeList } from 'react-window'

// Memoize expensive renders
const ExpensiveList = React.memo(({ items }) => (
  <FixedSizeList height={600} itemCount={items.length} itemSize={80} width="100%">
    {({ index, style }) => <Item item={items[index]} style={style} />}
  </FixedSizeList>
))

// Use transitions for non-urgent updates (React 18+)
import { useTransition } from 'react'

function SearchResults() {
  const [isPending, startTransition] = useTransition()
  const [query, setQuery] = useState('')

  function handleChange(e) {
    startTransition(() => setQuery(e.target.value))
  }
  // ...
}
```

## Hack/PHP → Modern JS Patterns

Meta's backend historically used Hack. These patterns carry over:

```ts
// Strict typing everywhere — Meta enforces Flow/TypeScript strictly
// No implicit any, no type assertions without justification

// Null safety — treat null and undefined as distinct error states
type Result<T> = { ok: true; value: T } | { ok: false; error: string }

function parseUser(data: unknown): Result<User> {
  if (!isUser(data)) return { ok: false, error: 'Invalid user data' }
  return { ok: true, value: data }
}

// Use discriminated unions over boolean flags
type RequestState =
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'success'; data: User }
  | { status: 'error'; message: string }
```

## Testing at Meta's Scale

### Test philosophy
- **Unit tests** for pure logic — fast, no I/O
- **Integration tests** for service boundaries
- **Snapshot tests** for UI components (Meta created Jest snapshot testing)
- **E2E tests** for critical user flows only

```tsx
// Jest + React Testing Library (Meta's testing stack)
import { render, screen, userEvent } from '@testing-library/react'

test('shows error message when login fails', async () => {
  server.use(
    rest.post('/api/login', (req, res, ctx) =>
      res(ctx.status(401), ctx.json({ error: 'Invalid credentials' }))
    )
  )

  render(<LoginForm />)
  await userEvent.type(screen.getByLabelText('Email'), 'user@example.com')
  await userEvent.type(screen.getByLabelText('Password'), 'wrongpassword')
  await userEvent.click(screen.getByRole('button', { name: 'Sign in' }))

  expect(await screen.findByText('Invalid credentials')).toBeInTheDocument()
})
```

### Snapshot testing
```tsx
// Use sparingly — for stable UI components, not dynamic content
test('renders UserCard correctly', () => {
  const { container } = render(<UserCard user={mockUser} />)
  expect(container).toMatchSnapshot()
})
// Update snapshots intentionally: jest --updateSnapshot
```

## Code Review Culture

Meta's review culture is fast and direct:
- **Ship fast, iterate** — don't block on perfection; ship and improve
- **Diff size** — keep diffs small and focused; large diffs get less thorough review
- **Inline comments** — be direct, specific, and constructive
- **Land and iterate** — it's better to land something good and improve it than to wait for perfect

### Diff description template
```
Summary: One sentence describing what changed.

Why: Why this change is needed.

How: Key implementation decisions.

Test Plan: How this was tested.

Reviewed by: @reviewer
```

## Observability & Instrumentation

Meta instruments everything. Apply this mindset:

```ts
// Log structured events, not strings
logger.info('user.login', {
  userId: user.id,
  method: 'email',
  duration_ms: Date.now() - startTime,
  success: true,
})

// Track performance metrics
performance.mark('render-start')
// ... render ...
performance.measure('component-render', 'render-start')

// Feature flags for gradual rollout (Meta's Gatekeeper pattern)
if (featureFlags.isEnabled('new_feed_algorithm', userId)) {
  return newFeedAlgorithm(userId)
}
return legacyFeedAlgorithm(userId)
```

## React Native (Meta's mobile stack)

```tsx
// Use StyleSheet.create for performance (styles are processed once)
const styles = StyleSheet.create({
  container: { flex: 1, backgroundColor: '#fff' },
  text: { fontSize: 16, color: '#333' },
})

// Avoid inline styles in render — they create new objects every render
// BAD:  <View style={{ flex: 1, padding: 16 }}>
// GOOD: <View style={styles.container}>

// Use FlatList, not ScrollView, for lists
<FlatList
  data={items}
  keyExtractor={item => item.id}
  renderItem={({ item }) => <ItemRow item={item} />}
  getItemLayout={(_, index) => ({ length: 80, offset: 80 * index, index })}
/>
```
