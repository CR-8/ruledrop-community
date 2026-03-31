Manage complex application state with Redux Toolkit (RTK). Use this skill when building large React apps with shared async state, complex update logic, or teams that need predictable, debuggable state flows.

## Store Setup

```ts
// store/store.ts
import { configureStore } from '@reduxjs/toolkit'
import postsReducer from '../features/posts/postsSlice'
import authReducer from '../features/auth/authSlice'

export const store = configureStore({
  reducer: {
    posts: postsReducer,
    auth: authReducer,
  },
})

export type RootState = ReturnType<typeof store.getState>
export type AppDispatch = typeof store.dispatch
```

```tsx
// main.tsx
import { Provider } from 'react-redux'
import { store } from './store/store'

ReactDOM.createRoot(document.getElementById('root')!).render(
  <Provider store={store}><App /></Provider>
)
```

## Typed Hooks

Always use typed hooks — never raw `useDispatch` or `useSelector`:

```ts
// store/hooks.ts
import { useDispatch, useSelector } from 'react-redux'
import type { RootState, AppDispatch } from './store'

export const useAppDispatch = () => useDispatch<AppDispatch>()
export const useAppSelector = <T>(selector: (state: RootState) => T): T =>
  useSelector(selector)
```

## Slice Pattern

One slice per feature. Keep slices focused — one concern per file.

```ts
// features/posts/postsSlice.ts
import { createSlice, PayloadAction } from '@reduxjs/toolkit'

interface Post { id: string; title: string; body: string; authorId: string }
interface PostsState {
  items: Post[]
  selectedId: string | null
  filter: string
}

const initialState: PostsState = {
  items: [],
  selectedId: null,
  filter: '',
}

const postsSlice = createSlice({
  name: 'posts',
  initialState,
  reducers: {
    postAdded: (state, action: PayloadAction<Post>) => {
      state.items.push(action.payload)  // Immer — mutate directly
    },
    postRemoved: (state, action: PayloadAction<string>) => {
      state.items = state.items.filter(p => p.id !== action.payload)
    },
    postSelected: (state, action: PayloadAction<string | null>) => {
      state.selectedId = action.payload
    },
    filterChanged: (state, action: PayloadAction<string>) => {
      state.filter = action.payload
    },
  },
})

export const { postAdded, postRemoved, postSelected, filterChanged } = postsSlice.actions
export default postsSlice.reducer
```

RTK uses Immer — you can write mutating logic inside reducers. It produces immutable updates under the hood.

## Async Thunks

Use `createAsyncThunk` for all async operations:

```ts
// features/posts/postsThunks.ts
import { createAsyncThunk } from '@reduxjs/toolkit'

export const fetchPosts = createAsyncThunk(
  'posts/fetchAll',
  async (_, { rejectWithValue }) => {
    try {
      const res = await fetch('/api/posts')
      if (!res.ok) throw new Error(`HTTP ${res.status}`)
      return await res.json() as Post[]
    } catch (err) {
      return rejectWithValue(err instanceof Error ? err.message : 'Failed to fetch')
    }
  }
)

export const createPost = createAsyncThunk(
  'posts/create',
  async (data: { title: string; body: string }, { rejectWithValue }) => {
    try {
      const res = await fetch('/api/posts', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(data),
      })
      if (!res.ok) throw new Error(`HTTP ${res.status}`)
      return await res.json() as Post
    } catch (err) {
      return rejectWithValue(err instanceof Error ? err.message : 'Failed to create')
    }
  }
)
```

Handle thunk lifecycle in `extraReducers`:

```ts
const postsSlice = createSlice({
  name: 'posts',
  initialState: {
    items: [] as Post[],
    status: 'idle' as 'idle' | 'loading' | 'succeeded' | 'failed',
    error: null as string | null,
  },
  reducers: { /* ... */ },
  extraReducers: (builder) => {
    builder
      .addCase(fetchPosts.pending, (state) => {
        state.status = 'loading'
        state.error = null
      })
      .addCase(fetchPosts.fulfilled, (state, action) => {
        state.status = 'succeeded'
        state.items = action.payload
      })
      .addCase(fetchPosts.rejected, (state, action) => {
        state.status = 'failed'
        state.error = action.payload as string
      })
  },
})
```

## Selectors

Define selectors alongside the slice. Use `createSelector` for derived/memoized values:

```ts
// features/posts/postsSelectors.ts
import { createSelector } from '@reduxjs/toolkit'
import type { RootState } from '../../store/store'

export const selectAllPosts = (state: RootState) => state.posts.items
export const selectPostsStatus = (state: RootState) => state.posts.status
export const selectFilter = (state: RootState) => state.posts.filter

// Memoized derived selector — only recomputes when inputs change
export const selectFilteredPosts = createSelector(
  [selectAllPosts, selectFilter],
  (posts, filter) =>
    filter ? posts.filter(p => p.title.toLowerCase().includes(filter.toLowerCase())) : posts
)

export const selectPostById = (id: string) =>
  createSelector(selectAllPosts, posts => posts.find(p => p.id === id))
```

## Usage in Components

```tsx
function PostList() {
  const dispatch = useAppDispatch()
  const posts = useAppSelector(selectFilteredPosts)
  const status = useAppSelector(selectPostsStatus)

  useEffect(() => { dispatch(fetchPosts()) }, [dispatch])

  if (status === 'loading') return <Spinner />
  if (status === 'failed') return <ErrorMessage />

  return (
    <ul>
      {posts.map(post => (
        <li key={post.id} onClick={() => dispatch(postSelected(post.id))}>
          {post.title}
        </li>
      ))}
    </ul>
  )
}
```

## RTK Query (Optional)

For API state, RTK Query eliminates thunks entirely:

```ts
// services/api.ts
import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/query/react'

export const api = createApi({
  reducerPath: 'api',
  baseQuery: fetchBaseQuery({ baseUrl: '/api' }),
  tagTypes: ['Post'],
  endpoints: (builder) => ({
    getPosts: builder.query<Post[], void>({ query: () => '/posts', providesTags: ['Post'] }),
    createPost: builder.mutation<Post, Partial<Post>>({
      query: (body) => ({ url: '/posts', method: 'POST', body }),
      invalidatesTags: ['Post'],
    }),
  }),
})

export const { useGetPostsQuery, useCreatePostMutation } = api
```

## Key Rules

- One slice per feature — keep them small and focused
- Always use typed hooks (`useAppDispatch`, `useAppSelector`)
- Define selectors in a separate file alongside the slice
- Use `createSelector` for any derived state to avoid unnecessary re-renders
- Use `rejectWithValue` in thunks to pass structured error data to the reducer
- Prefer RTK Query over manual thunks for straightforward CRUD operations
