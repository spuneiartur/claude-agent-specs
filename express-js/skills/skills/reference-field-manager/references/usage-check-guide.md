# Usage Check Guide

The `functions/check-usage.js` file provides a centralized way to check if a resource is referenced by other models before allowing deletion. This prevents orphaned references.

## How It Works

`checkUsage(entityType, entityId, options)` looks up the entity type in `USAGE_CONFIGS`, queries all configured models, and throws an error if the entity is in use.

## USAGE_CONFIGS Structure

```js
const USAGE_CONFIGS = {
  {entityType}: {
    entityName: '{EntityName}',
    entityNameForError: 'the {entity}',
    modelChecks: [
      {
        modelName: '{ReferencingModel}',
        checks: [
          {
            queryField: '{refField}._id',
            displayName: '{Referencing Models}',
            description: 'as {entity} for {referencing models}',
          },
        ],
      },
    ],
  },
};
```

### Fields Explained

- **`entityType`** — key used to look up the config (matches the first arg to `checkUsage`)
- **`entityName`** — display name for the entity in error messages
- **`entityNameForError`** — lowercase version for error context
- **`modelChecks`** — array of models to check
  - **`modelName`** — PascalCase model name (must match export name in `models/index.js`)
  - **`checks`** — array of query patterns for this model
    - **`queryField`** — the MongoDB query field path (e.g. `'category._id'`, `'tags._id'`)
    - **`displayName`** — human-readable name for the referencing collection
    - **`description`** — describes the relationship for error messages

## Real Example: Category

```js
category: {
  entityName: 'Category',
  entityNameForError: 'the category',
  modelChecks: [
    {
      modelName: 'Product',
      checks: [
        {
          queryField: 'category._id',
          displayName: 'Products',
          description: 'as category for products',
        },
      ],
    },
  ],
},
```

## Real Example: Image (multiple referencing models)

```js
image: {
  entityName: 'Image',
  entityNameForError: 'the image',
  modelChecks: [
    {
      modelName: 'Texture',
      checks: [
        {
          queryField: 'swatch._id',
          displayName: 'Textures',
          description: 'as swatch for textures',
        },
      ],
    },
    {
      modelName: 'ProductVariant',
      checks: [
        {
          queryField: 'images._id',
          displayName: 'Product Variants',
          description: 'as image for product variants',
        },
      ],
    },
  ],
},
```

## Using checkUsage in Delete Controllers

```js
import { checkUsage } from '@functions';
import { Category } from '@models';

export default async (req, res) => {
  const { id } = req.params;

  // Check if category is used before deleting
  await checkUsage('category', id);

  await Category.findByIdAndDelete(id);

  return res.status(200).json({
    data: req.model,
    message: 'Category deleted successfully!',
  });
};
```

If the entity is in use, `checkUsage` throws an error like:
```
"Category used in: Products (3)"
```

## Options

```js
// Default: throws error if used
await checkUsage('category', id);

// Don't throw, return usage info instead
const result = await checkUsage('category', id, { throwError: false });
// result: { isUsed: true, usageResults: [...], totalReferences: 3 }

// Include detailed field info
const result = await checkUsage('category', id, { throwError: false, includeDetails: true });
```
