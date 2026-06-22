# Claude Code Instructions

## Tech stack

- React Native 0.85+ (New Architecture enabled)
- Expo SDK 56+ (latest) with `expo-router` (file-based routing)
- React 19.2
- `react-native-unistyles` v3 for styling (NOT StyleSheet from react-native, NOT NativeWind/Tailwind)
- `react-native-reanimated` v4 — requires the `react-native-worklets/plugin` Babel plugin (last plugin)
- `@tanstack/react-query` v5 for server state, wrapped by `hooks/use-query`, `use-mutation`, `use-infinite-query`
- `zustand` for client/auth state, persisted via `expo-secure-store`
- `react-hook-form` + `yup` for forms
- `axios` for HTTP (two clients: public + authenticated)
- `expo-router` typed routes
- No TypeScript
- No Redux
- No tests

## Commands

- `npm start` / `yarn start` — lint then start Expo dev server (`--dev-client`)
- `npm run start-nolint` — start without linting
- `npm run ios` / `npm run android` — build and run native (`expo run:*`)
- `npm run web` — start web build
- `npm run lint` / `npm run lint-fix` — ESLint
- `npm run format` — Prettier
- `npm run setup-ios` / `setup-android` / `setup-all` — `expo prebuild`
- `npm run reset` — wipe node_modules, reinstall, reset Metro cache

## Naming conventions

- All file/folder names: kebab-case (`auth-store.js`, `is-jwt-valid.js`)
- `.jsx` React component files: PascalCase (`ButtonPrimary.jsx`, `ThemedText.jsx`)
- Route files in `app/`: lowercase, expo-router conventions (`login.jsx`, `[id].jsx`, `_layout.jsx`)
- Route groups: parentheses `(auth)`, `(protected)`, `(tabs)` — do NOT affect the URL
- Private/core building blocks: underscore prefix (`_Button.jsx`, `_Input.jsx`, `_core/`)

## Folder structure

- `app` — expo-router routes (file-based). Layouts (`_layout.jsx`), groups (`(auth)`), screens. App entry is `app/_layout.jsx`
- `app/examples` — reference screens (check here first for working patterns!)
- `api` — backend request modules, grouped by subject (`auth.js`). Functions call `Axios`/`AxiosAuth`, surface errors via `Toaster`
- `assets` — fonts, images, animations (Lottie JSON)
- `components` — shared React Native components + barrel `index.js`
- `components/ui` — design-system primitives (Buttons, Inputs, PageContainers, NavigationBars, TabNavigators), each with a `_core/` internal implementation
- `components/Forms` — react-hook-form system (`HookForm`, `Form`, `Field`, `Fieldset`, `Submit`)
- `constants` — app constants
- `constants/theme` — unistyles theme tokens (`colors`, `spacing`, `typography`, `borderRadius`, `breakpoints`) + `unistyles.js` config
- `functions` — pure utility functions (`is-jwt-valid.js`, `normalize.js`, `extract-error.js`, `is-request-canceled.js`)
- `hooks` — custom hooks: `useTheme.js` + React Query wrappers (`use-query.js`, `use-mutation.js`, `use-infinite-query.js`)
- `lib` — configured library wrappers + barrel `index.js` (axios clients, zustand auth store, secure/async storage, toaster)

## Path aliases

Use `@aliases` instead of relative paths. Configured in BOTH `babel.config.js` (module-resolver) and `jsconfig.json`:
`@components`, `@lib`, `@services`, `@assets`, `@hooks`, `@functions`, `@constants`, `@app`

> `api/` is imported as `api/auth` (no alias) — match the existing import style in the file you are editing.

## Coding rules

- **Check `app/examples/` first** for similar working implementations
- Do NOT create new files/folders in the project root
- Each file ends with an empty line
- Order imports and environment variables A-Z
- Use ES6 imports, never `require` (exception: `require('../assets/...')` for static asset resolution and `require('app.json')`)
- One default export per file, except barrel `index.js` files (named re-exports)
- **Write short files — max 60-70 lines of code per file**, single responsibility
- Use comments and empty lines to delimit groups of code
- Do NOT nest code more than 2 levels deep
- Functions: max 4 parameters, max ~50 executable lines, lines ≤ 80 chars
- Always use optional chaining (`?.`)
- Do NOT add new NPM packages unless explicitly instructed
- Search for existing primitives in `components/ui/` before using raw RN elements
- Use `<ThemedText>` instead of RN `<Text>` — do NOT style ThemedText directly; wrap in a `<View>` and style that
- Use theme spacing tokens, not magic numbers: `theme.spacing.md` not `16`

## Styling rules (react-native-unistyles v3)

- Style with `StyleSheet.create((theme, rt) => ({ ... }))` imported from `react-native-unistyles` (NOT from `react-native`)
- `theme` exposes `colors`, `spacing`, `typography`, `borderRadius` — use tokens, not literals
- `rt` exposes runtime info: `rt.insets.top` for safe-area padding
- Dynamic styles: return a function from a style key — `container: (isDisabled) => ({ ... })`, call as `styles.container(disabled)`
- Light/dark themes are defined in `constants/theme/`; toggle via `useTheme()` / `UnistylesRuntime.setTheme()`
- `constants/theme/unistyles.js` MUST be imported first — `index.js` does this before `expo-router/entry`
- Do NOT introduce NativeWind/Tailwind or `react-native` `StyleSheet`

## Routing rules (expo-router)

- File-based: a file in `app/` becomes a route; `_layout.jsx` defines a navigator (`Stack`, `Tabs`)
- Route groups `(name)` organize without affecting the URL — use for `(auth)`, `(protected)`, `(tabs)`
- Protected routes: `app/(protected)/_layout.jsx` checks `useAuthStore` and `<Redirect href='/login' />` when unauthenticated
- Dynamic routes use bracket notation: `[id].jsx`, read params with `useLocalSearchParams`
- Navigate with `router.push`/`router.back` from `expo-router` or `useRouter()`
- Typed routes are enabled (`app.json` → `experiments.typedRoutes`)

## API & auth rules

- Two axios clients in `lib/`: `Axios` (public, `@lib`) and `AxiosAuth` (injects Bearer token, handles 401 → logout)
- Response interceptors return `res.data` directly and throw `err.response.data` — `api/` functions receive unwrapped data
- Auth state lives in `useAuthStore` (zustand + persist + secure-store adapter): `{ token, user, isOnboarded }`
- `api/` functions are try/catch wrapped, set store state via `useAuthStore.setState`, and report UX via `Toaster.success`/`Toaster.error`; return `null` on failure
- Validate tokens with `functions/is-jwt-valid.js`
- Secrets/config via `process.env.EXPO_PUBLIC_*` (see `.env.example`)

## Data fetching rules (TanStack Query)

- A single `QueryClient` is created in `app/_layout.jsx` with `queryClientConfig` from `constants/query-client.js` and provided via `QueryClientProvider`
- **Fetch with the hook wrappers, not raw axios in screens**: `useQuery(url, params, options)`, `useInfiniteQuery(url, params, options)` for paginated lists, and `useMutation(fn, options)` for writes — all from `@hooks`
- `useQuery`/`useInfiniteQuery` call `AxiosAuth.get` under the hood; `params` are run through `normalize()` to build a stable `queryKey` and the request query
- `useMutation` auto-toasts `data.message` on success and the extracted message on error; options: `invalidateQueries`, `redirectOnSuccess`, `successCallback`, `errorCallback`
- Canceled requests (stale-query aborts) are detected via `isRequestCanceled` and never retried or toasted
- Keep `api/` modules for imperative one-off calls (auth) and as `mutationFn`s; use the query hooks for screen data

## Forms rules

- Build forms with the `components/Forms` system: `HookForm` (provider + yup resolver) → `Form` (layout, optional `debug`) → `Field` (wraps an input via `as=`) → `Submit`
- Validation schemas use `yup` (`Yup.object().shape({...})`)
- `Field` connects to react-hook-form `Controller`; pass the input component via the `as` prop and `name`/`label`/`help`
- Submit handler signature: `(values, methods) => {}` — use `methods.reset()` after success

## Skills

Development pattern skills are in `skills/skills/`. Key skills:

- `screen-scaffold` — create expo-router screens (public, protected, tabs, dynamic, layouts)
- `component-factory` — create RN components with unistyles + barrel exports + `_core` pattern
- `form-builder` — react-hook-form + yup using the `Forms` system
- `api-service` — create `api/` request modules with `Axios`/`AxiosAuth`, store, and `Toaster`
- `data-fetching` — fetch/mutate screen data with the TanStack Query hooks (`useQuery`/`useMutation`/`useInfiniteQuery`)
- `theme-styling` — unistyles styling, theme tokens, light/dark theming
- `skill-creator` — create new skills from scratch
