Build production-grade Next.js applications using the App Router. Use this skill when creating pages, layouts, server components, server actions, data fetching patterns, or structuring a Next.js 13+ project.


## Core Mental Model

The App Router has two component types. Get this right first — everything else follows.

- **Server Components** (default) — render on the server, can `async/await`, can access databases and secrets directly, zero JS sent to client
- **Client Components** (`'use client'`) — run in the browser, can use hooks, event handlers, browser APIs

**Rule:** push `'use client'` as far down the tree as possible. A server component can import a client component, not the other way around.

## File Conventions

```
app/
  layout.tsx          ← root layout, always a server component
  page.tsx            ← route segment UI
  loading.tsx         ← automatic Suspense boundary
  error.tsx           ← error boundary ('use client' required)
  not-found.tsx       ← 404 UI
  route.ts            ← API route handler
  (group)/            ← route group, no URL segment
  [slug]/             ← dynamic segment
  [...slug]/          ← catch-all segment
```

## Data Fetching

Fetch directly in server components. No `useEffect`, no API layer needed for internal data.

```tsx
// app/posts/page.tsx — server component
async function PostsPage() {
  const posts = await db.post.findMany({ orderBy: { createdAt: 'desc' } })
  return <PostList posts={posts} />
}
```

**Parallel fetching** — avoid waterfalls by initiating requests together:

```tsx
async function Dashboard() {
  const [user, stats, feed] = await Promise.all([
    getUser(),
    getStats(),
    getFeed(),
  ])
  return <DashboardUI user={user} stats={stats} feed={feed} />
}
```

**Caching** — Next.js caches `fetch()` by default. Be explicit:

```tsx
// Cache indefinitely (static)
fetch(url, { cache: 'force-cache' })

// Never cache (dynamic, always fresh)
fetch(url, { cache: 'no-store' })

// Revalidate every N seconds (ISR)
fetch(url, { next: { revalidate: 60 } })
```

## Server Actions

Use server actions for mutations. No separate API route needed for form submissions.

```tsx
// app/actions.ts
'use server'

import { revalidatePath } from 'next/cache'

export async function createPost(formData: FormData) {
  const title = formData.get('title') as string
  if (!title?.trim()) throw new Error('Title is required')

  await db.post.create({ data: { title } })
  revalidatePath('/posts')
}
```

```tsx
// app/posts/new/page.tsx
import { createPost } from '../actions'

export default function NewPostPage() {
  return (
    <form action={createPost}>
      <input name="title" required />
      <button type="submit">Create</button>
    </form>
  )
}
```

For progressive enhancement and pending state, use `useActionState` (React 19) or `useFormStatus`.

## Layouts and Nested Routing

Layouts persist across navigations — don't put per-page state in them.

```tsx
// app/dashboard/layout.tsx
export default function DashboardLayout({ children }: { children: React.ReactNode }) {
  return (
    <div className="dashboard">
      <Sidebar />
      <main>{children}</main>
    </div>
  )
}
```

Use route groups `(group)` to share layouts without affecting the URL:

```
app/
  (marketing)/
    layout.tsx   ← marketing layout
    page.tsx     ← /
    about/page.tsx ← /about
  (app)/
    layout.tsx   ← app layout (auth-gated)
    dashboard/page.tsx ← /dashboard
```

## Loading and Streaming

`loading.tsx` automatically wraps the page in a `<Suspense>` boundary.

For granular streaming, use `<Suspense>` directly around slow components:

```tsx
export default function Page() {
  return (
    <>
      <HeroSection />           {/* renders immediately */}
      <Suspense fallback={<FeedSkeleton />}>
        <SlowFeed />            {/* streams in when ready */}
      </Suspense>
    </>
  )
}
```

## Metadata

Use the `metadata` export or `generateMetadata` for dynamic SEO:

```tsx
// Static
export const metadata = {
  title: 'My App',
  description: 'Description here',
}

// Dynamic
export async function generateMetadata({ params }: { params: { slug: string } }) {
  const post = await getPost(params.slug)
  return {
    title: post.title,
    openGraph: { images: [post.coverImage] },
  }
}
```

## Route Handlers

Use `route.ts` for API endpoints (webhooks, third-party callbacks, public APIs):

```ts
// app/api/webhook/route.ts
import { NextRequest, NextResponse } from 'next/server'

export async function POST(req: NextRequest) {
  const body = await req.json()
  // handle webhook
  return NextResponse.json({ received: true })
}
```

Prefer server actions over route handlers for internal mutations.

## Key Rules

- Never use `getServerSideProps` or `getStaticProps` — those are Pages Router patterns
- Avoid `'use client'` on layouts and pages unless absolutely necessary
- Use `next/image` for all images — automatic optimization, lazy loading, size hints
- Use `next/link` for all internal navigation — prefetching is automatic
- Colocate components with their route segment unless they're shared
- Use `revalidatePath` or `revalidateTag` after mutations to bust the cache
- For any more context refer to `https://nextjs.org/docs`
