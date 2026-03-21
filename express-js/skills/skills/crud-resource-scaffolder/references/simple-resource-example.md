# Simple Resource Example: `tag`

A standalone resource with no references to other models. This is the base template for all simple CRUD resources.

## File 1: `models/tag.js`

```js
import { validate } from 'express-goodies/middleware';
import { paginate, softDelete } from 'express-goodies/mongoose';
import trails from 'express-goodies/mongoose/trails';
import { model, Schema } from 'mongoose';

const name = 'tag';
const schema = new Schema(
  {
    name: {
      type: String,
      required: true,
      unique: true,
    },
  },
  { timestamps: true }
);

schema.plugin(paginate);
schema.plugin(validate);
schema.plugin(softDelete);
schema.plugin(trails);

export default model(name, schema);
```

## File 2: `schemas/schema-tag.js`

```js
import * as yup from 'yup';
import safeSchema from './safe-schema';

const tagSchema = safeSchema({
  name: yup.string().lowercase().trim().required('Name is required'),
});

export default tagSchema;
```

## File 3: `filters/apply-filters-tag.js`

```js
const applyFiltersTag = (query) => {
  const { search = '', created_from, created_to } = query;

  const filter = {};

  if (search) {
    filter.name = { $regex: search, $options: 'i' };
  }

  if (created_from) {
    filter.createdAt = { $gte: created_from };
  }
  if (created_to) {
    filter.createdAt = { ...filter.createdAt, $lt: created_to };
  }

  return filter;
};

export default applyFiltersTag;
```

## File 4: `controllers/tag/create.js`

```js
import { error } from '@functions';
import { Tag } from '@models';

export default async (req, res) => {
  const { name } = req.body;

  const alreadyExists = await Tag.findOne({ name });
  if (alreadyExists) {
    throw error(400, 'A tag with this name already exists');
  }

  const document = await Tag.create(req.body);

  return res.status(200).json({
    data: document,
    message: 'Tag created successfully!',
  });
};
```

## File 5: `controllers/tag/read-one.js`

```js
import { error } from '@functions';
import { Tag } from '@models';

export default async (req, res) => {
  const { id } = req.params;

  const tag = await Tag.findById(id);

  if (!tag) {
    throw error(404, 'Tag not found');
  }

  return res.status(200).json(tag);
};
```

## File 6: `controllers/tag/read-many.js`

```js
import { applyFiltersTag } from '@filters';
import { Tag } from '@models';

export default async (req, res) => {
  const documents = await Tag.find(applyFiltersTag(req.query)).paginate(req.query);

  return res.status(200).json(documents);
};
```

## File 7: `controllers/tag/update.js`

```js
import { error } from '@functions';
import { Tag } from '@models';

export default async (req, res) => {
  const { id } = req.params;
  const { name } = req.body;

  const alreadyExists = await Tag.findOne({ name, _id: { $ne: id } });
  if (alreadyExists) {
    throw error(400, 'A tag with this name already exists');
  }

  const updated = await Tag.findByIdAndUpdate(id, req.body, { new: true });

  return res.status(200).json({
    data: updated,
    message: 'Tag updated successfully!',
  });
};
```

**Note:** If this resource is referenced by other models (has trail functions), add trail update call:
```js
import { error, updateTagTrails } from '@functions';
// ... after findByIdAndUpdate:
await updateTagTrails(updated);
```

## File 8: `controllers/tag/delete.js`

```js
import { Tag } from '@models';

export default async (req, res) => {
  const { id } = req.params;

  await Tag.findByIdAndDelete(id);

  return res.status(200).json({
    data: req.model,
    message: 'Tag deleted successfully!',
  });
};
```

**Note:** If this resource is referenced by other models (has trail functions), add trail remove call:
```js
import { removeTagTrails } from '@functions';
// ... after findByIdAndDelete:
await removeTagTrails(id);
```

## File 9: `controllers/tag/index.js`

```js
export { default as create } from './create';
export { default as deleteTag } from './delete';
export { default as readMany } from './read-many';
export { default as readOne } from './read-one';
export { default as update } from './update';
```

**Important:** The delete export is named `delete{Resource}` (e.g. `deleteTag`) since `delete` is a reserved word.

## File 10: `routes/tag.js`

```js
import { Tag } from '@controllers';
import { validate } from '@middleware';
import tagSchema from '@schemas/schema-tag';
import { Router } from 'express';

const router = Router();
export default router;

router.get('/admin/tags', Tag.readMany);
router.get('/admin/tags/:id', Tag.readOne);
router.post('/admin/tags', validate(tagSchema), Tag.create);
router.put('/admin/tags/:id', validate(tagSchema), Tag.update);
router.delete('/admin/tags/:id', Tag.deleteTag);
```

## Optional File 11: `models/schemas/ref-tag.js` (only if referenced by other models)

Not applicable for tag in this example — tag uses a simpler ref pattern where it's stored as an object with `_id` and `name` directly in the referencing model's schema. See `ref-category.js` in the relational example for the full ref schema pattern.

## Optional File 12: `schemas/ref-tag.js` (only if referenced by other models)

```js
import * as yup from 'yup';

const tagRef = yup.object().shape({
  _id: yup.string().required('Tag ID is required'),
  name: yup.string().required('Tag name is required'),
});

export default tagRef;
```
