---
name: public-endpoint-builder
description: Create public (unauthenticated) API endpoints for existing Express.js + MongoDB resources. Generates list and detail controllers with aggregation pipelines, and wires them into the public routes file. Use this skill whenever the user wants to add a public endpoint, create a storefront/frontend API, expose a resource publicly, add a public route, or says things like "I need a public page for products" or "create a frontend API for articles" or "add a public list endpoint for reviews".
---

# Public Endpoint Builder

Creates public (unauthenticated) read endpoints for existing resources. Public endpoints live in `controllers/public/` and `routes/public.js`, separate from admin CRUD. They often use aggregation pipelines, populate related data, and have different response shapes than admin endpoints.

## When to Use

- "Add a public endpoint for products"
- "Create a frontend API for articles"
- "I need a public page route for product details"
- "Expose reviews publicly"
- "Add a storefront list endpoint"

## Key Differences from Admin CRUD

| Aspect | Admin CRUD | Public Endpoints |
|--------|-----------|-----------------|
| Auth | Protected by `middleware.authenticate` | No auth (under `/public/` prefix which gets speed limiter) |
| Location | `controllers/{resource}/` | `controllers/public/` |
| Routes | `routes/{resource}.js` | `routes/public.js` |
| Operations | Full CRUD | Read-only (GET) |
| Data shape | Direct model queries | Often aggregation pipelines with `$lookup` |
| Filtering | Admin filters | May add `isActive: true` or similar |

## Naming Convention

Read `references/naming-guide.md` for full details.

- **`get-{resources}.js`** — lightweight list endpoint (for dropdowns, selectors, simple lists)
- **`view-page-{resources}.js`** — full list page with pagination, filtering, complex data
- **`view-page-{resource}.js`** — single resource detail page (by slug, with populated relations)

## Workflow

### Step 1: Interview

Gather from the user:

1. **Which resource** to expose publicly?
2. **What type of endpoint?**
   - List (paginated, filtered) → `view-page-{resources}` or `get-{resources}`
   - Detail (single item by slug/id) → `view-page-{resource}`
   - Both
3. **What data does the frontend need?** — just the base document, or populated relations too?
4. **Any filtering?** — active-only, category-based, search, etc.

### Step 2: Generate Controllers

Read `references/public-patterns.md` for complete examples.

#### Simple List — `controllers/public/get-{resources}.js`

For lightweight lists (dropdowns, simple lists). Uses basic `.find()`:

```js
import { {Resource} } from '@models';

export default async (req, res) => {
  const documents = await {Resource}.find({ isActive: true }).select('name slug').lean();

  return res.status(200).json(documents);
};
```

#### Full List Page — `controllers/public/view-page-{resources}.js`

For paginated list pages with filtering. Uses aggregation pipeline:

```js
import { applyFilters{Resource} } from '@filters';
import { {Resource} } from '@models';

export default async (req, res) => {
  const page = parseInt(req.query.page, 10) || 1;
  const perPage = parseInt(req.query.perPage, 10) || 10;

  const filters = {
    ...applyFilters{Resource}(req.query),
    isActive: true,
  };

  const totalCount = await {Resource}.countDocuments(filters);

  const documents = await {Resource}.aggregate([
    { $match: filters },
    // Add $lookup stages for related data if needed
    { $skip: (page - 1) * perPage },
    { $limit: perPage },
  ]);

  const pageParams = {
    count: totalCount,
    hasNext: page * perPage < totalCount,
    page,
    perPage,
  };

  return res.status(200).json({ pageParams, pages: documents });
};
```

#### Detail Page — `controllers/public/view-page-{resource}.js`

For single resource pages (by slug). Uses aggregation to populate relations:

```js
import { error } from '@functions';
import { {Resource} } from '@models';

export default async (req, res) => {
  const { slug } = req.params;

  if (!slug) {
    throw error(400, 'Slug is required');
  }

  const result = await {Resource}.aggregate([
    { $match: { slug } },
    // Add $lookup stages for related data
    // Add $addFields for computed fields
    // Add $project to exclude temporary fields
  ]);

  if (result.length === 0) {
    throw error(404, '{Resource} not found');
  }

  return res.status(200).json(result[0]);
};
```

### Common Aggregation Patterns

#### Populating a ref field via `$lookup`

```js
{
  $lookup: {
    from: '{collection_name}',        // MongoDB collection name (usually plural + underscore)
    localField: '{refField}._id',
    foreignField: '_id',
    as: '{refField}Data',
  },
},
{
  $addFields: {
    {refField}: {
      $mergeObjects: ['${refField}', { $arrayElemAt: ['${refField}Data', 0] }],
    },
  },
},
```

#### Cleaning up temporary fields

```js
{
  $project: {
    {refField}Data: 0,
  },
},
```

### Step 3: Update Public Controller Barrel

In `controllers/public/index.js`, add the new exports in A-Z order:

```js
export { default as get{Resources} } from './get-{resources}';
export { default as viewPage{Resource} } from './view-page-{resource}';
export { default as viewPage{Resources} } from './view-page-{resources}';
```

### Step 4: Add Routes

In `routes/public.js`, add routes under a comment section for the resource:

```js
// {Resources}
router.get('/public/get-{resources}', Public.get{Resources});
router.get('/public/view-page-{resources}', diacriticInsensitive(['search']), Public.viewPage{Resources});
router.get('/public/view-page-{resource}/:slug', Public.viewPage{Resource});
```

Add `diacriticInsensitive(['search'])` to list endpoints that support text search.

## Important Rules

- All public endpoints are **read-only** (GET only)
- No authentication middleware — public routes are under `/public/` prefix
- Add `isActive: true` filter when the model has an `isActive` field
- Use aggregation pipelines for endpoints that need populated data
- Return response shapes that match what the frontend expects
- Keep English messages: `'{Resource} not found'`, `'Slug is required'`

## Reference Files

- `references/public-patterns.md` — Real public controller examples from the codebase
- `references/naming-guide.md` — When to use `get-X` vs `view-page-X` naming
