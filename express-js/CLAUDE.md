# CLAUDE.md

## Project Overview

Express.js + MongoDB REST API built from a custom starter template. Node ES6, no TypeScript, no Docker.

## Commands

- `npm run dev` — start dev server (nodemon, port 9000)
- `npm run build` — Babel transpile to `dist/`
- `npm run start` — start production build
- `npm run seed` — seed database
- `npm test` — run mocha tests
- `node npm.js {script-name}` — run a one-off script from `scripts/`

## Project Structure

```
controllers/   CRUD handlers, organized by resource (e.g. controllers/tag/)
models/        Mongoose schemas and models
models/schemas/ Embedded reference schemas (ref-category.js, ref-tag.js)
schemas/       Yup validation schemas (schema-*.js) and ref schemas (ref-*.js)
routes/        Express route definitions
filters/       Query filter builders (apply-filters-*.js)
functions/     Utility helpers (error, trails, usage checks)
middleware/    Custom middleware (validate, normalizeDropdownValues, diacriticInsensitive)
constants/     Enum-like constant values
plugins/       Third-party integrations (AWS S3, Postmark, Netopia, SmartBill)
db/            Seeds and resources for initial data
scripts/       One-off import/migration scripts
docs/          Project documentation and wiki
examples/      Reference implementation (Todo CRUD)
```

## Path Aliases

Use `@aliases` instead of relative paths for cross-directory imports:
`@controllers`, `@models`, `@functions`, `@filters`, `@schemas`, `@middleware`, `@plugins`, `@routes`, `@db`

Configured in `jsconfig.json` and `babel.config.json`.

## Coding Standards

- ES6 imports ordered A-Z. No `require`.
- One default export per file (except barrel `index.js` files)
- Max 40-50 lines per file — single responsibility
- No `try/catch` in controllers — uses `express-async-errors`
- Return `200` HTTP status on success (not 201/204)
- End controllers with `return` statements
- Use `throw error(statusCode, message)` from `@functions` for errors
- Use return-early pattern
- Do NOT add new npm packages unless explicitly instructed
- Do NOT create new files/folders in the project root

## Naming Conventions

- Files/folders: kebab-case (`blog-comment`, `product-variant`)
- CRUD keywords: `list`, `view`, `create`, `update`, `remove`
- Controller files: `create.js`, `read-one.js`, `read-many.js`, `update.js`, `delete.js`
- Route paths: `/admin/{resources}` (plural) for admin CRUD, `/public/{endpoint}` for public

## API Response Patterns

- **Errors**: `{ name, message }`
- **Mutations** (create/update/delete): `{ data, message }`
- **GET many**: `{ pages, pageParams: { count, hasNext, page, perPage } }`
- **GET one**: direct payload (no wrapper keys)

## Key Patterns

- **Models**: Mongoose with plugins — `paginate`, `validate`, `softDelete`, `trails`
- **Validation**: Yup schemas via `validate()` middleware, built with `safeSchema()`
- **Refs**: Embedded reference objects (`{ _id, name, slug }`) with Mongoose + Yup ref schemas
- **Trails**: When a referenced doc changes, trail functions propagate updates to all embedders
- **Usage checks**: `checkUsage()` in `functions/check-usage.js` prevents deleting referenced docs
- **normalizeDropdownValues**: Middleware that fetches full docs by ID for ref fields on POST/PUT
- **diacriticInsensitive**: Middleware for accent-tolerant search on GET routes

## Documentation

Detailed guides are in `docs/`:
- `docs/coding-standards.md` — full coding rules
- `docs/naming-conventions.md` — file and keyword conventions
- `docs/wiki/how-to-return-data-from-the-api.md` — response format examples
- `docs/wiki/how-to-filter-query-data.md` — filter implementation guide
- `docs/wiki/how-to-secure-your-api.md` — auth, recaptcha, rate limiting
- `docs/wiki/how-to-use-normalize-dropdown-values.md` — dropdown middleware guide
