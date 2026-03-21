---
name: crud-resource-scaffolder
description: Scaffold a complete CRUD resource for an Express.js + MongoDB API. Generates all files — model, validation schema, filter, controllers, routes, and barrel exports — in one shot. Use this skill whenever the user wants to add a new resource, create CRUD endpoints, scaffold an entity, add a new model, or says things like "I need a new collection for X" or "create endpoints for X". Even if the user only mentions a model or a single endpoint, this skill ensures the full resource is scaffolded correctly.
---

# CRUD Resource Scaffolder

Scaffolds a complete CRUD admin resource following the project's Express.js + MongoDB patterns. One resource = ~12 files across 7 directories + updates to 6 barrel index files and the main router.

## When to Use

Any time the user wants to add a new entity/resource/collection to the API. This includes requests like:
- "Add a new resource called review"
- "Create CRUD for testimonials"
- "I need a model for FAQ with question and answer fields"
- "Scaffold a blog-comment entity"

## Workflow

### Step 1: Interview

Gather from the user:

1. **Resource name** — singular, kebab-case (e.g. `review`, `blog-comment`, `faq-item`)
2. **Fields** — for each field: name, type (String/Number/Boolean/Date/Array/Object), required?, unique?
3. **Relationships** — does this resource reference other models? (e.g. "review references product")
4. **Unique constraint field** — which field is checked for duplicates on create/update? (typically `name` or `slug`)

### Step 2: Classify Complexity

- **Simple resource** (like `tag`): no refs to other models, basic fields. Read `references/simple-resource-example.md`.
- **Relational resource** (like `product`): has embedded refs to other models, needs `normalizeDropdownValues` middleware. Read `references/relational-resource-example.md`.

### Step 3: Generate Files

Generate ALL files below. Read `references/coding-standards.md` for rules that apply to every file.

For a resource named `{resource}` (singular, kebab-case) with PascalCase name `{Resource}`:

#### 3a. Model — `models/{resource}.js`

```js
import { validate } from 'express-goodies/middleware';
import { paginate, softDelete } from 'express-goodies/mongoose';
import trails from 'express-goodies/mongoose/trails';
import { model, Schema } from 'mongoose';

const name = '{resource}';
const schema = new Schema(
  {
    // fields here
  },
  { timestamps: true }
);

schema.plugin(paginate);
schema.plugin(validate);
schema.plugin(softDelete);
schema.plugin(trails);

export default model(name, schema);
```

#### 3b. Yup Validation Schema — `schemas/schema-{resource}.js`

```js
import * as yup from 'yup';
import safeSchema from './safe-schema';

const {resource}Schema = safeSchema({
  // field validations here using yup
});

export default {resource}Schema;
```

Use `yup.string()`, `yup.number()`, `yup.boolean()`, `yup.date()`, `yup.array()`, `yup.object()`. Add `.required('Field is required')` for required fields. Add `.trim()` for strings. Use camelCase for the schema variable name.

#### 3c. Filter — `filters/apply-filters-{resource}.js`

```js
const applyFilters{Resource} = (query) => {
  const { search = '', created_from, created_to } = query;

  const filter = {};

  if (search) {
    filter.{searchField} = { $regex: search, $options: 'i' };
  }

  if (created_from) {
    filter.createdAt = { $gte: created_from };
  }
  if (created_to) {
    filter.createdAt = { ...filter.createdAt, $lt: created_to };
  }

  return filter;
};

export default applyFilters{Resource};
```

Replace `{searchField}` with the primary searchable field (usually `name` or `title`).

#### 3d. Controllers — `controllers/{resource}/`

Create 5 files + 1 barrel index. See `references/simple-resource-example.md` for exact templates.

- **`create.js`** — Check duplicate on unique field, `Model.create(req.body)`, return `{ data, message }`
- **`read-one.js`** — `Model.findById(id)`, throw 404 if not found, return document
- **`read-many.js`** — `Model.find(applyFilters(req.query)).paginate(req.query)`, return documents
- **`update.js`** — Check duplicate (excluding self with `$ne: id`), `findByIdAndUpdate(id, req.body, { new: true })`, return `{ data, message }`
- **`delete.js`** — `Model.findByIdAndDelete(id)`, return `{ data: req.model, message }`
- **`index.js`** — Barrel file exporting all controllers

The delete controller export name should be `delete{Resource}` (e.g. `deleteReview`) since `delete` is a reserved word.

#### 3e. Routes — `routes/{resource}.js`

**Simple resource:**
```js
import { {Resource} } from '@controllers';
import { validate } from '@middleware';
import {resource}Schema from '@schemas/schema-{resource}';
import { Router } from 'express';

const router = Router();
export default router;

router.get('/admin/{resources}', {Resource}.readMany);
router.get('/admin/{resources}/:id', {Resource}.readOne);
router.post('/admin/{resources}', validate({resource}Schema), {Resource}.create);
router.put('/admin/{resources}/:id', validate({resource}Schema), {Resource}.update);
router.delete('/admin/{resources}/:id', {Resource}.delete{Resource});
```

**Relational resource** (has refs): additionally import `diacriticInsensitive`, `normalizeDropdownValues`, and referenced models. See `references/relational-resource-example.md`.

### Step 4: Update Barrel Exports

Read `references/barrel-exports-guide.md` for the exact export patterns. Update these files maintaining **A-Z alphabetical order**:

1. **`controllers/index.js`** — `export * as {Resource} from './{resource}';`
2. **`models/index.js`** — `export { default as {Resource} } from './{resource}';`
3. **`routes/index.js`** — `export { default as {resource} } from './{resource}';`
4. **`filters/index.js`** — `export { default as applyFilters{Resource} } from './apply-filters-{resource}';`
5. **`schemas/index.js`** — Add under `// ===== VALIDATION SCHEMAS =====` section: `export { default as {resource}Schema } from './schema-{resource}';`

### Step 5: Register Route in Router

In `router.js`, add `router.use(routes.{resource});` in A-Z order among the other `router.use(routes.xxx)` lines, **before** `router.use(routes.public)` and `router.use(exampleRoutes.todo)`.

### Step 6: Optional — Reference Schemas

Only if this resource will be embedded as a ref in other models:

- Create `models/schemas/ref-{resource}.js` — Mongoose embedded ref schema. See `references/simple-resource-example.md` for the pattern.
- Create `schemas/ref-{resource}.js` — Yup ref validation schema.
- Add ref exports to `schemas/index.js` under `// ===== REFERENCE SCHEMAS =====`.

## Important Rules

- All messages in English: `'{Resource} created successfully!'`, `'{Resource} not found'`, `'A {resource} with this {field} already exists'`
- Always use `@aliases` for imports (`@models`, `@controllers`, `@functions`, `@filters`, `@schemas`, `@middleware`)
- One default export per file (except barrel `index.js` files)
- Max 40-50 lines per file
- No `try/catch` in controllers — the app uses `express-async-errors`
- Return HTTP `200` on success for all operations
- End every controller with a `return` statement
- Use `error()` from `@functions` for throwing errors
- Order imports alphabetically (A-Z)
- No new npm packages

## Reference Files

- `references/simple-resource-example.md` — Complete `tag` resource as template (all 12 files)
- `references/relational-resource-example.md` — Complete `product` resource showing relational patterns
- `references/coding-standards.md` — All coding rules for generated files
- `references/barrel-exports-guide.md` — How each barrel index.js uses different export patterns
