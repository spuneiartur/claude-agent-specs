---
name: api-service
description: >
  Create backend request modules in the `api/` folder using the starter's two axios clients (public
  `Axios` and authenticated `AxiosAuth`), the zustand auth store, and `Toaster` for UX feedback. Use
  this skill whenever the user wants to call an API, add a backend request, wire up authentication
  (login/logout), fetch or mutate server data, add an endpoint call, or handle API errors. Trigger on:
  "call the API", "fetch data", "add an endpoint", "login/logout", "make a request", "wire up auth",
  "POST/GET to the backend", or any client-server communication.
---

# API Service

Backend calls live in `api/`, grouped by subject (`api/auth.js`, `api/items.js`). They use two
preconfigured axios clients from `@lib` and surface results through `Toaster`.

## The two clients

| Client | Import | Use for |
|--------|--------|---------|
| `Axios` | `@lib` | public/unauthenticated endpoints |
| `AxiosAuth` | `@lib` | authenticated endpoints — injects `Authorization: Bearer <token>` and on 401 auto-logs-out |

Both share `axiosBaseHeaders` (platform, device, app version, client type) and **response
interceptors that return `res.data` directly and throw `err.response.data`**. So in an `api/`
function, the awaited value is already the unwrapped payload — do NOT read `.data` again.

```js
const user = await AxiosAuth.get('/me');   // user is the payload, not an axios response
```

`baseURL` comes from `process.env.EXPO_PUBLIC_API_BASE_URL`.

## Auth store

Auth state is a zustand store (`useAuthStore` from `@lib`), persisted to secure storage:
`{ token, user, isOnboarded }`. Read it reactively in components, imperatively in `api/`:

```js
import { useAuthStore } from '@lib';

const token = useAuthStore.getState().token;        // imperative read
useAuthStore.setState({ token, user });              // imperative write
const isAuthed = useAuthStore((s) => !!s.token);     // reactive (in a component)
```

## Pattern: a mutation that updates auth state

`api/auth.js` (the canonical example):

```js
import { Axios, Toaster, useAuthStore } from '@lib';
import { router } from 'expo-router';
import { jwtDecode } from 'jwt-decode';

export const login = async (email, password) => {
    try {
        const data = await Axios.post('/login', { email, password });

        const { token, user } = data;
        if (!jwtDecode(token)) {
            throw new Error('Invalid token');
        }

        useAuthStore.setState({
            token,
            user,
            isOnboarded: user?.hasOnboarded ?? false,
        });

        Toaster.success('Logged in');
        return data;
    } catch (error) {
        console.error('Login error:', error);
        Toaster.error(error.message || error || 'Login failed');
        return null;
    }
};

export const logout = async () => {
    useAuthStore.setState({ token: null, user: null, isOnboarded: false });
    Toaster.success('Logged out');
    router.push('/login');
    return true;
};
```

## Pattern: a typical resource module

`api/items.js`:

```js
import { AxiosAuth, Toaster } from '@lib';

export const listItems = async (params) => {
    try {
        return await AxiosAuth.get('/admin/items', { params });
    } catch (error) {
        Toaster.error(error?.message || 'Failed to load items');
        return null;
    }
};

export const createItem = async (payload) => {
    try {
        const data = await AxiosAuth.post('/admin/items', payload);
        Toaster.success('Item created');
        return data;
    } catch (error) {
        Toaster.error(error?.message || 'Failed to create item');
        return null;
    }
};
```

## Calling from a screen

Prefer the TanStack Query hooks over manual `useEffect` fetching — see the **`data-fetching`** skill.
Use `api/` functions as the `mutationFn` for writes:

```jsx
import { useMutation } from '@hooks';
import { createItem } from 'api/items';

const mutation = useMutation(createItem, { invalidateQueries: ['/admin/items'] });
mutation.mutate({ name: 'New item' });
```

For reads, call `useQuery('/admin/items')` directly (it uses `AxiosAuth` internally) rather than wiring
a `useState`/`useEffect` around an `api/` function.

> `api/` modules are imported by path (`api/auth`, `api/items`), matching existing usage — there is
> no `@api` alias.

## Rules

- One subject per file in `api/`; named exports (verbs: `login`, `listItems`, `createItem`, `updateItem`, `removeItem`)
- Use `AxiosAuth` for anything requiring a token; `Axios` for public endpoints
- Each function is wrapped in try/catch, reports via `Toaster.success`/`Toaster.error`, and returns
  `null` on failure so callers can guard with `if (!data) return;`
- Do NOT re-unwrap `.data` — the interceptor already returned the payload
- Validate JWTs with `functions/is-jwt-valid.js` when needed
- Mutate auth state only via `useAuthStore.setState`
- Imports ordered A-Z; keep files short; trailing empty line
- Never hardcode secrets — use `process.env.EXPO_PUBLIC_*`
