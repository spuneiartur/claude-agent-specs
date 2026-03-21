# Barrel Exports Guide

Each barrel `index.js` file uses a specific export pattern. Follow these exactly.

## `controllers/index.js` — namespace exports

```js
export * as Review from './review';
```

Imports all named exports from the controller's `index.js` as a namespace object. This allows `Review.create`, `Review.readOne`, etc.

## `models/index.js` — default re-exports

```js
export { default as Review } from './review';
```

Re-exports each model's default export (the Mongoose model) as a named export.

## `routes/index.js` — default re-exports (camelCase)

```js
export { default as review } from './review';
```

**Note:** camelCase name, matching the variable name used in `router.js`.

## `filters/index.js` — default re-exports (camelCase)

```js
export { default as applyFiltersReview } from './apply-filters-review';
```

## `schemas/index.js` — two sections

```js
// ===== REFERENCE SCHEMAS =====
export { default as reviewRef } from './ref-review';

// ===== VALIDATION SCHEMAS =====
export { default as reviewSchema } from './schema-review';
```

Reference schemas go in the first section; validation schemas in the second. Only add a ref schema if the resource is referenced by other models.

## `functions/index.js` — mixed sources

```js
// Re-exports from express-goodies (don't touch these)
export { default as error } from 'express-goodies/functions/error';
// ...

// Custom functions (add new ones here in A-Z order)
export { default as updateReviewTrails } from './update-review-trails';
export { default as removeReviewTrails } from './remove-review-trails';
```

Only add trail functions if the resource is embedded as a ref in other models.

## Ordering Rule

All exports in every barrel file MUST be in **A-Z alphabetical order** by export name. When inserting a new export, find its correct alphabetical position.
