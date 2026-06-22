---
name: data-fetching
description: >
  Fetch and mutate server data in React Native screens using the starter's TanStack Query (React Query
  v5) hook wrappers — useQuery, useInfiniteQuery, and useMutation from @hooks, backed by AxiosAuth. Use
  this skill whenever the user wants to load data into a screen, show a list, paginate/infinite-scroll,
  create/update/delete via the API, cache server state, invalidate/refetch after a mutation, or replace
  manual useState+useEffect+axios fetching. Trigger on: "fetch data", "load the list", "useQuery",
  "react query", "tanstack query", "mutation", "infinite scroll", "pagination", "invalidate", "refetch",
  or any screen that reads/writes server state.
---

# Data Fetching (TanStack Query)

Server state uses **@tanstack/react-query v5** through thin wrappers in `hooks/`. **Do not call axios
directly in screens** — use these hooks so caching, cancellation, retries, and toasts are consistent.

A single `QueryClient` is created in `app/_layout.jsx` from `constants/query-client.js`
(`queryClientConfig`) and provided via `QueryClientProvider`. Hooks are exported from `@hooks`.

## The hooks

| Hook | Signature | Use for |
|------|-----------|---------|
| `useQuery` | `useQuery(url, params?, options?)` | reading a resource / list |
| `useInfiniteQuery` | `useInfiniteQuery(url, params?, options?)` | paginated / infinite-scroll lists |
| `useMutation` | `useMutation(fn, options?)` | create / update / delete |

`useQuery` and `useInfiniteQuery` call `AxiosAuth.get(url, { params, signal })`. `params` are passed
through `normalize()` (in `@functions`) to drop empty values and build a **stable `queryKey`** — so key
order never causes cache misses. Because the axios interceptor already unwraps `res.data`, `data` is the
payload directly.

## Reading data

```jsx
import { useQuery } from '@hooks';

const { data: items, status, isFetching, refetch } = useQuery('/admin/items', { search });

// status: 'pending' | 'error' | 'success'
```

Render the three states explicitly (don't rely on `data` being defined):

```jsx
{status === 'pending' && <ActivityIndicator />}
{status === 'error' && <ThemedText>Failed to load.</ThemedText>}
{status === 'success' && items?.map((item) => <Row key={item.id} item={item} />)}
```

Pass any React Query option through the third arg: `useQuery(url, params, { enabled: !!id, staleTime: 0 })`.

## Paginated lists

`useInfiniteQuery` expects the API's `{ pages, pageParams: { hasNext, page, perPage } }` shape and
flattens every page into a single `pages` array via `select`:

```jsx
import { useInfiniteQuery } from '@hooks';

const { data, fetchNextPage, hasNextPage, isFetchingNextPage } = useInfiniteQuery('/admin/items');

<FlatList
    data={data?.pages}
    keyExtractor={(item) => String(item.id)}
    renderItem={({ item }) => <Row item={item} />}
    onEndReached={() => hasNextPage && fetchNextPage()}
    onEndReachedThreshold={0.4}
/>
```

## Mutations

`useMutation(fn, options)` — `fn` is any promise-returning function (often an `api/` service). It
auto-toasts `data.message` on success and the extracted error message on failure.

```jsx
import { useMutation } from '@hooks';
import { createItem } from 'api/items';

const mutation = useMutation(createItem, {
    invalidateQueries: ['/admin/items'],   // refetch the list after success
    redirectOnSuccess: '/items',           // optional expo-router push
    successCallback: (data) => {},         // optional
    errorCallback: (err) => {},            // optional
});

// trigger it
mutation.mutate({ name: 'New item' });
// state: mutation.isPending, mutation.isError
```

`invalidateQueries` takes a `queryKey` prefix — pass the same `url` you used in `useQuery`/`useInfiniteQuery`
(React Query matches by prefix). Canceled requests are ignored (no retry, no toast) via `isRequestCanceled`.

## When to use what

- **Screen data (read)** → `useQuery` / `useInfiniteQuery`. Replaces `useState` + `useEffect` + axios.
- **Writes** → `useMutation` with an `api/` function as `mutationFn` (see the `api-service` skill).
- **Imperative one-offs** (login, logout, token refresh) → call the `api/` function directly.

## Rules

- No raw axios in screens — go through the hooks (or an `api/` service used as a `mutationFn`)
- `queryKey` is derived from `[url, normalize(params).key]` — don't hand-build keys; just pass `params`
- Always handle `status === 'pending' | 'error' | 'success'`
- Invalidate the list query after create/update/delete with `invalidateQueries`
- Tune caching per-call via the options arg, not by editing the global client unless it's a global change
- See `app/examples/tanstack-query.jsx` for a working screen
