# Claude Code Instructions

## Tech stack

- React 18+
- Next.js 15+ (Pages Router, NOT App Router)
- Tailwind CSS
- No Docker
- No TypeScript
- No CSS-in-JS
- No global state (React Query for server state, Context for UI state, minimal Redux for auth only)
- No tests

## Naming conventions

- All file names must be lowercase
- `.jsx` React component files must use PascalCase syntax
- All other files must use kebab-case syntax

## Folder structure

- `api` — backend requests, grouped by subject
- `auth` — authentication logic (JWT, withAuth HOC, checkAuth)
- `components` — React components, grouped by subject
- `constants` — constant variables: months, countries, etc.
- `css` — custom CSS files
- `data` — project data: table columns, etc.
- `docs` — documentation
- `examples` — example implementations (check here first!)
- `functions` — pure utility functions
- `hooks` — custom React hooks
- `languages` — translations (JSON)
- `lib` — library wrappers with custom configuration (axios, toaster, logger, classnames)
- `models` — Yup validation schemas for forms
- `pages` — Next.js pages (browser routes)
- `public` — static assets: images, icons, documents
- `skills` — Claude agent skills for development patterns
- `headers.js` — security headers
- `site.config.js` — project configuration

## Coding rules

- **Check `examples/` folder first** for similar implementations
- Do NOT create new files and folders in the project root
- Order imports and environment variables by name (A-Z)
- Use ES6 imports (not `require`) in all new files
- Use only one default export per file, except `index.js` barrel files
- **Write short files, maximum 40-50 lines of code per file**
- Use comments and empty lines to delimit groups of code
- Use `@aliases` instead of relative paths: `@api`, `@auth`, `@components`, `@constants`, `@css`, `@data`, `@functions`, `@hooks`, `@lib`, `@models`, `@site.config`
- Do NOT add new NPM packages unless explicitly instructed
- Always search for existing custom components first (Button.jsx, Image.jsx, etc.) before using raw HTML elements
- For data fetching use `hooks/use-query.js`
- For mutations use `hooks/use-mutation.js`
- For paginated lists use `hooks/use-infinite-query.js`

## Table pattern

For admin tables, follow the pattern from `examples/components/Todos/TodoTable.jsx`:

- Use `components/Tables/` system: `TableColumns`, `TableLoading`, `TableError`, `TableSuccess`
- Use `hooks/use-infinite-query.js` for paginated data fetching
- Use `components/Buttons/LoadMoreOnClick` or `LoadMoreOnScroll` for pagination
- Define column configs in `data/` folder using TanStack React Table format (`id`, `header`, `accessorKey`, optional `cell` component)
- Handle all 3 states: `pending` → `TableLoading`, `error` → `TableError`, `success` → `TableSuccess`
- Use `Bone` component for loading skeletons (via `extraClass` on columns for width hints)

## Design rules

- Use Tailwind CSS utility classes for styling
- Do NOT use arbitrary value syntax like `p-[50px]`
- Prefer custom CSS (in `css/` folder) when an element needs more than ~10 utility classes
- Use fewer `<div>` elements
- Use Font Awesome icons: `<i className="fas fa-bars"></i>`
- Search for existing components before creating new ones
- **Separate status components**: use `{Entity}Loading.jsx`, `{Entity}Error.jsx`, `{Entity}Success.jsx` for `useQuery` states. Use `Bone` component for loading skeletons.
- Component variants use CSS classes from `css/` files, e.g. `<Button className="button full primary">`

## Next.js rules

- Use dynamic routes with bracket notation `[id].js`
- Prefer flat, descriptive routes
- Do NOT use the `use client` directive
- Public pages use `PresentationLayout`, admin pages use `Layout` with `checkAuth`/`withAuth`

## Skills

Development pattern skills are in `skills/skills/`. Key skills:

- `admin-crud-scaffold` — scaffold full CRUD for a new admin entity
- `form-builder` — React Hook Form + Yup with HookForm/Field system
- `api-hook-wiring` — connect API services with React Query hooks
- `page-scaffold` — create new public or admin pages
- `component-factory` — create components following conventions
- `custom-hook` — create custom hooks
- `website-speed-audit` — performance audit and image/video optimization
- `seo-strategy` — SEO content optimization + Next.js SEO implementation
