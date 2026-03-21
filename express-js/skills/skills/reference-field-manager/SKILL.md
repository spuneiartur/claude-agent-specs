---
name: reference-field-manager
description: Manage embedded reference fields between Express.js + MongoDB resources. Creates Mongoose ref schemas, Yup ref schemas, trail update/remove functions, usage check configs, and wires up normalizeDropdownValues middleware. Use this skill whenever the user wants to add a relationship between models, embed a reference, link two resources, add a dropdown field that references another model, or says things like "product should reference category" or "add author to articles". This skill prevents the data integrity bugs that come from missing any part of the reference lifecycle.
---

# Reference Field Manager

Handles the full lifecycle of adding an embedded reference field between two resources in the Express.js + MongoDB API. When resource A references resource B, there are 5-7 files to create/modify — missing any one causes data integrity bugs that are hard to trace.

## When to Use

- "Product should reference category"
- "Add a relationship between X and Y"
- "Embed a ref to author in articles"
- "Add a dropdown field for texture in product variants"
- "Link reviews to products"

## Terminology

- **Referencing model** (A): the model that contains the embedded ref field (e.g. Product)
- **Referenced model** (B): the model being referenced (e.g. Category)
- **Trails**: when B changes, all documents in A that embed B's data need to be updated

## Workflow

### Step 1: Interview

Gather:
1. **Referencing model** (A) — which model gets the new ref field?
2. **Referenced model** (B) — which model is being referenced?
3. **Fields to embed** — which fields from B to store in A? (typically `_id`, `name`, and optionally `slug`)
4. **Is the relationship required?** — must A always have a ref to B?
5. **Is it a single ref or array of refs?** — e.g. one category vs many tags

### Step 2: Create/Verify Schemas

Read `references/ref-schema-examples.md` for complete patterns.

#### 2a. Mongoose Ref Schema — `models/schemas/ref-{referenced}.js`

Only create if it doesn't already exist. Check first.

```js
import { Types } from 'mongoose';

const {referenced}ModelRef = {
  _id: {
    type: Types.ObjectId,
    ref: '{referenced}',
    required: true,
  },
  name: {
    type: String,
    required: true,
  },
  // add other embedded fields as needed
};

export default {referenced}ModelRef;
```

#### 2b. Yup Ref Schema — `schemas/ref-{referenced}.js`

Only create if it doesn't already exist.

```js
import * as yup from 'yup';

const {referenced}Ref = yup.object().shape({
  _id: yup.string().required('{Referenced} ID is required'),
  name: yup.string().required('{Referenced} name is required'),
});

export default {referenced}Ref;
```

### Step 3: Update the Referencing Model

In `models/{referencing}.js`:
1. Import the Mongoose ref schema: `import {referenced}ModelRef from './schemas/ref-{referenced}';`
2. Add the field to the schema:
   - **Single ref**: `{referenced}: {referenced}ModelRef`
   - **Array of refs**: `{referenced}s: [{referenced}ModelRef]`

### Step 4: Update the Validation Schema

In `schemas/schema-{referencing}.js`:
1. Import the Yup ref schema: `import {referenced}Ref from './ref-{referenced}';`
2. Add the field validation:
   - **Single ref**: `{referenced}: {referenced}Ref.required('{Referenced} is required')`
   - **Array of refs**: `{referenced}s: yup.array().of({referenced}Ref)`

### Step 5: Create Trail Functions

Read `references/trail-examples.md` for complete patterns.

#### 5a. Update trails — `functions/update-{referenced}-trails.js`

Propagates changes when the referenced document is updated.

```js
import { {Referencing} } from '../models';

const update{Referenced}Trails = async ({referenced}) => {
  for (const Model of [{Referencing}]) {
    await Model.updateMany(
      { '{referenced}._id': {referenced}?._id },
      {
        '{referenced}.$.name': {referenced}?.name,
        // update other embedded fields
      }
    );
  }
};

export default update{Referenced}Trails;
```

For array refs, use the `$` positional operator as shown above. For single refs, use direct field assignment:
```js
await Model.updateMany(
  { '{referenced}._id': {referenced}?._id },
  {
    '{referenced}.name': {referenced}?.name,
  }
);
```

#### 5b. Remove trails — `functions/remove-{referenced}-trails.js`

Cleans up when the referenced document is deleted.

```js
import { {Referencing} } from '../models';

const remove{Referenced}Trails = async ({referenced}Id) => {
  for (const Model of [{Referencing}]) {
    await Model.updateMany(
      { '{referenced}._id': {referenced}Id },
      { $pull: { {referenced}s: { _id: {referenced}Id } } }
    );
  }
};

export default remove{Referenced}Trails;
```

For single refs (not arrays), set the field to null instead of using `$pull`:
```js
await Model.updateMany(
  { '{referenced}._id': {referenced}Id },
  { '{referenced}': null }
);
```

### Step 6: Wire Trail Functions into Controllers

In `controllers/{referenced}/update.js`, add the trail update call after `findByIdAndUpdate`:
```js
import { update{Referenced}Trails } from '@functions';
// ... after update:
await update{Referenced}Trails(updated);
```

In `controllers/{referenced}/delete.js`, add the trail remove call after `findByIdAndDelete`:
```js
import { remove{Referenced}Trails } from '@functions';
// ... after delete:
await remove{Referenced}Trails(id);
```

### Step 7: Add Usage Check Config

Read `references/usage-check-guide.md` for the full pattern.

In `functions/check-usage.js`, add a new entry to `USAGE_CONFIGS`:

```js
{referenced}: {
  entityName: '{Referenced}',
  entityNameForError: 'the {referenced}',
  modelChecks: [
    {
      modelName: '{Referencing}',
      checks: [
        {
          queryField: '{referenced}._id',
          displayName: '{Referencing}s',
          description: 'as {referenced} for {referencing}s',
        },
      ],
    },
  ],
},
```

Then call `checkUsage` in `controllers/{referenced}/delete.js` before deleting:
```js
import { checkUsage } from '@functions';
await checkUsage('{referenced}', id);
```

### Step 8: Wire normalizeDropdownValues in Routes

In `routes/{referencing}.js`, add the middleware to POST and PUT routes:

```js
import { normalizeDropdownValues, validate } from '@middleware';
import { {Referenced} } from '@models';

router.post(
  '/admin/{referencing}s',
  normalizeDropdownValues([{ field: '{referenced}', model: {Referenced} }]),
  validate({referencing}Schema),
  {Referencing}.create
);
router.put(
  '/admin/{referencing}s/:id',
  normalizeDropdownValues([{ field: '{referenced}', model: {Referenced} }]),
  validate({referencing}Schema),
  {Referencing}.update
);
```

If there are already existing `normalizeDropdownValues` entries, add to the existing array.

### Step 9: Update Barrel Exports

- `schemas/index.js` — add ref schema under `// ===== REFERENCE SCHEMAS =====`
- `functions/index.js` — add trail function exports in A-Z order

## Reference Files

- `references/ref-schema-examples.md` — Mongoose + Yup ref schemas side by side
- `references/trail-examples.md` — Update + remove trail function examples
- `references/usage-check-guide.md` — How USAGE_CONFIGS works in check-usage.js
