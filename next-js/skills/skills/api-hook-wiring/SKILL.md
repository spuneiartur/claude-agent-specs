---
name: api-hook-wiring
description: >
  Create API service modules and wire them up with React Query hooks (useQuery, useMutation,
  useInfiniteQuery) for data fetching, mutations, and paginated lists. Use this skill whenever
  the user needs to connect a frontend feature to a backend API, add data fetching to a page or
  component, set up mutations for form submissions, create API service functions, add infinite
  scrolling, or wire up any client-server data flow. Trigger on: "fetch data", "API call",
  "connect to backend", "mutation", "infinite scroll", "paginated list", "data fetching",
  "useQuery", "useMutation", or when the user describes any feature that reads or writes data.
---

# API Hook Wiring

Connect frontend features to backend APIs using the project's Axios + React Query system. The project provides custom hooks that wrap React Query with automatic URL building, toast notifications, query invalidation, and router redirects.

## Architecture

```
API Service (api/*.js)          →  defines HTTP calls using axios instances
    ↓
Custom Hooks (hooks/*.js)       →  wraps React Query with project defaults
    ↓
Components                      →  consume hooks for data + mutations
```

## Step 1: Create the API Service

Create `api/{resource}.js`. Choose the right axios instance:

- **`axiosAuth`** from `@lib` — for authenticated endpoints (admin, user-specific)
- **`axios`** from `@lib` — for public endpoints (no auth header)

```js
import { axiosAuth } from '@lib';

// Read operations
export const get{Resource} = (id) => axiosAuth.get(`/admin/{resources}/${id}`);
export const get{Resources} = (params) => axiosAuth.get('/admin/{resources}', { params });

// Write operations
export const create{Resource} = (data) => axiosAuth.post('/admin/{resources}', data);
export const update{Resource} = ({ id, data }) => axiosAuth.put(`/admin/{resources}/${id}`, data);
export const delete{Resource} = (id) => axiosAuth.delete(`/admin/{resources}/${id}`);
```

For public endpoints:
```js
import { axios } from '@lib';

export const getPublic{Resources} = () => axios.get('/public/{resources}');
```

## Step 2: Wire Up Data Fetching

### useQuery — Single resource or list

```jsx
import { useQuery } from '@hooks';

// Fetch by ID (URL is both the endpoint and the cache key)
const { data: resource, status } = useQuery(`admin/{resources}/${id}`);

// Fetch with query params (options become URL query params)
const { data, status } = useQuery('admin/{resources}', { status: 'active', page: 1 });
```

How `useQuery` works internally:
- Builds URL using `query-string`: `stringifyUrl({ url, query: options })`
- Uses `axiosAuth` for internal URLs, plain `axios` for external URLs (starting with `http`)
- The full URL becomes the React Query cache key

### useInfiniteQuery — Paginated lists with load-more

```jsx
import { useInfiniteQuery } from '@hooks';

const { data, status, ...paginationProps } = useInfiniteQuery('admin/{resources}', options);
```

How `useInfiniteQuery` works internally:
- Default `per_page: 30`
- Merges `options` as query params
- Auto-handles `getNextPageParam` from `pageParams.hasNext`
- Flattens response: `data.pages` is an array of page arrays, `data.pageParams` has pagination metadata

Use with table components:
```jsx
<TableColumns pageParams={data?.pageParams} />
{status === 'loading' && <TableLoading name="{resources}" columns={columns} />}
{status === 'error' && <TableError name="{resources}" columns={columns} />}
{status === 'success' && (
  <>
    <TableSuccess name="{resources}" columns={columns} data={data} {...paginationProps} />
    <LoadMoreOnClick {...paginationProps} />
  </>
)}
```

## Step 3: Wire Up Mutations

### useMutation — Create, update, delete

```jsx
import { useMutation } from '@hooks';
import { createResource } from '@api/resource';

const mutation = useMutation(createResource, {
  invalidateQueries: 'admin/{resources}',  // refetch list after mutation
  successCallback: () => router.push('/admin/{resources}'),  // redirect on success
  // errorCallback: () => {},  // optional error handler
  // redirectOnSuccess: '/admin/{resources}',  // alternative to successCallback
});

// Trigger the mutation
const handleSubmit = async (data) => mutation.mutateAsync(data);
```

How `useMutation` works internally:
- On success: invalidates specified queries, shows toast with `data.message`, redirects, calls callback
- On error: shows toast with `err.message`, calls error callback
- Returns standard React Query mutation object (`mutateAsync`, `isPending`, `isError`, etc.)

For **update mutations**, the API function receives `{ id, data }`:
```jsx
const mutation = useMutation(updateResource, { invalidateQueries: 'admin/{resources}' });
const handleSubmit = async (data) => mutation.mutateAsync({ id: resource._id, data });
```

For **delete mutations**:
```jsx
const mutation = useMutation(deleteResource, { invalidateQueries: 'admin/{resources}' });
const handleDelete = async () => { await mutation.mutateAsync(id); };
```

## Status-Based Rendering Pattern

Always handle all three states when using `useQuery` or `useInfiniteQuery`:

```jsx
const { data, status } = useQuery(url);

{status === 'loading' && <LoadingComponent />}
{status === 'error' && <ErrorComponent />}
{status === 'success' && <SuccessComponent data={data} />}
```

## Axios Instances

The project has two axios instances in `lib/`:

| Instance | File | Auth | Use For |
|----------|------|------|---------|
| `axiosAuth` | `lib/axios-auth.js` | Bearer token from Redux store | Admin and user endpoints |
| `axios` | `lib/axios.js` | None | Public endpoints |

Both instances:
- Auto-extract `res.data` via response interceptor
- Handle errors with toast notifications
- `axiosAuth` additionally handles 401 → token refresh → retry flow

## Server-Side Data Fetching

For pages that need data at render time, use `getServerSideProps`:

```jsx
export async function getServerSideProps(context) {
  try {
    const data = await publicApi.getResources();
    return { props: { data } };
  } catch {
    return { props: { data: [] } };
  }
}
```

For admin pages, combine with auth check:
```jsx
export async function getServerSideProps(context) {
  return await checkAuth(context);
}
```

## Key Import Map

| What | Import From |
|------|------------|
| useQuery, useInfiniteQuery, useMutation | `@hooks` |
| axiosAuth, axios | `@lib` |
| TableColumns, TableLoading, TableError, TableSuccess | `@components/Tables` |
| LoadMoreOnClick | `@components/Buttons` |
| checkAuth | `@auth` |
