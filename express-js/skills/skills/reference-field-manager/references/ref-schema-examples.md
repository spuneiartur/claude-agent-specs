# Reference Schema Examples

Side-by-side examples of Mongoose ref schemas and their corresponding Yup ref schemas.

## Example 1: Category (with slug)

### Mongoose — `models/schemas/ref-category.js`

```js
import { Types } from 'mongoose';

const categoryModelRef = {
  _id: {
    type: Types.ObjectId,
    ref: 'category',
    required: true,
  },
  name: {
    type: String,
    required: true,
  },
  slug: {
    type: String,
    required: true,
  },
};

export default categoryModelRef;
```

### Yup — `schemas/ref-category.js`

```js
import * as yup from 'yup';

const categoryRef = yup.object().shape({
  _id: yup.string().required('Category ID is required'),
  name: yup.string().required('Category name is required'),
  slug: yup.string().required('Category slug is required'),
});

export default categoryRef;
```

## Example 2: Tag (minimal — name only)

### Mongoose — (inline in referencing model)

Tags don't have a separate ref schema file in this codebase — they're stored as an array of `{ _id, name }` objects directly. But if creating one:

```js
import { Types } from 'mongoose';

const tagModelRef = {
  _id: {
    type: Types.ObjectId,
    ref: 'tag',
    required: true,
  },
  name: {
    type: String,
    required: true,
  },
};

export default tagModelRef;
```

### Yup — `schemas/ref-tag.js`

```js
import * as yup from 'yup';

const tagRef = yup.object().shape({
  _id: yup.string().required('Tag ID is required'),
  name: yup.string().required('Tag name is required'),
});

export default tagRef;
```

## Example 3: Gallery Image (with URL fields)

### Mongoose — `models/schemas/ref-gallery.js`

```js
import { Types } from 'mongoose';

const galleryModelRef = {
  _id: {
    type: Types.ObjectId,
    ref: 'gallery',
    required: true,
  },
  title: {
    type: String,
    required: true,
  },
  url: {
    type: String,
    required: true,
  },
};

export default galleryModelRef;
```

## Patterns

1. **Always include `_id`** with `Types.ObjectId` and `ref` pointing to the model name string
2. **Include commonly needed display fields** — usually `name` and/or `slug`
3. **Mirror fields** between Mongoose and Yup schemas — same field names, same required status
4. **Yup `_id`** is always `yup.string()` (ObjectIds serialize to strings in JSON)
5. **Variable naming**: `{resource}ModelRef` for Mongoose, `{resource}Ref` for Yup

## Using Refs in Models

**Single ref:**
```js
import authorModelRef from './schemas/ref-author';

const schema = new Schema({
  author: authorModelRef,
});
```

**Array of refs:**
```js
import tagModelRef from './schemas/ref-tag';

const schema = new Schema({
  tags: [tagModelRef],
});
```

## Using Refs in Yup Schemas

**Single ref:**
```js
import authorRef from './ref-author';

const articleSchema = safeSchema({
  author: authorRef.required('Author is required'),
});
```

**Array of refs:**
```js
import tagRef from './ref-tag';

const articleSchema = safeSchema({
  tags: yup.array().of(tagRef),
});
```
