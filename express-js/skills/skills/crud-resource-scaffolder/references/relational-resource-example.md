# Relational Resource Example: `product`

A resource that references other models (Category, ProductType). This shows the additional patterns needed when a resource has embedded references.

## Key Differences from Simple Resource

1. **Model** includes imported ref schemas for related models
2. **Route** uses `normalizeDropdownValues` and `diacriticInsensitive` middleware
3. **Controllers** validate related documents exist and handle derived fields
4. **Additional files**: Mongoose ref schemas and Yup ref schemas

## File 1: `models/product.js` (excerpt — key patterns)

```js
import { validate } from 'express-goodies/middleware';
import { paginate, softDelete } from 'express-goodies/mongoose';
import trails from 'express-goodies/mongoose/trails';
import { model, Schema } from 'mongoose';
import categoryModelRef from './schemas/ref-category';
import productTypeModelRef from './schemas/ref-product-type';

const name = 'product';
const schema = new Schema(
  {
    title: { type: String, required: true },
    slug: { type: String, required: true, unique: true },
    category: categoryModelRef,           // embedded ref
    productType: productTypeModelRef,     // embedded ref
    isActive: { type: Boolean, default: true },
    // ... other fields
  },
  { timestamps: true }
);

schema.plugin(paginate);
schema.plugin(validate);
schema.plugin(softDelete);
schema.plugin(trails);

export default model(name, schema);
```

**Pattern:** Import ref schemas from `./schemas/ref-{resource}` (relative path since it's within the same `models/` directory).

## File 2: `models/schemas/ref-category.js` (Mongoose ref schema)

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

**Pattern:** Always includes `_id` with `Types.ObjectId` and `ref` pointing to the model name. Include the most commonly needed fields (typically `name` and/or `slug`).

## File 3: `schemas/ref-category.js` (Yup ref schema)

```js
import * as yup from 'yup';

const categoryRef = yup.object().shape({
  _id: yup.string().required('Category ID is required'),
  name: yup.string().required('Category name is required'),
  slug: yup.string().required('Category slug is required'),
});

export default categoryRef;
```

## File 4: `schemas/schema-product.js` (uses ref schemas)

```js
import * as yup from 'yup';
import categoryRef from './ref-category';
import safeSchema from './safe-schema';

const productSchema = safeSchema({
  title: yup.string().trim().required('Title is required'),
  slug: yup.string().trim().required('Slug is required'),
  category: categoryRef.required('Category is required'),
  isActive: yup.boolean(),
});

export default productSchema;
```

**Pattern:** Import ref schemas and use them as field validators. Add `.required()` on the ref if the relationship is mandatory.

## File 5: `routes/product.js` (with middleware chain)

```js
import { Product } from '@controllers';
import { diacriticInsensitive, normalizeDropdownValues, validate } from '@middleware';
import { Category } from '@models';
import { productSchema } from '@schemas';
import { Router } from 'express';

const router = Router();
export default router;

router.get('/admin/products', diacriticInsensitive(['search']), Product.readMany);
router.get('/admin/products/:id', Product.readOne);
router.post(
  '/admin/products',
  normalizeDropdownValues([{ field: 'category', model: Category }]),
  validate(productSchema),
  Product.create
);
router.put(
  '/admin/products/:id',
  normalizeDropdownValues([{ field: 'category', model: Category }]),
  validate(productSchema),
  Product.update
);
router.delete('/admin/products/:id', Product.deleteProduct);
```

### Middleware Details

**`diacriticInsensitive(['search'])`** — Converts diacritics in specified query params for accent-tolerant search. Use on GET many routes when the resource has text fields that might contain accented characters.

**`normalizeDropdownValues([{ field, model }])`** — Fetches the full document from MongoDB by ID for each specified field. The frontend sends only the `_id`, and this middleware fetches and attaches the full embedded ref object. Use on POST/PUT routes for every field that references another model.

Configuration array format:
```js
normalizeDropdownValues([
  { field: 'category', model: Category },
  { field: 'author', model: Author },
  // add one entry per ref field
])
```

## File 6: `controllers/product/create.js` (validates related docs)

```js
import { error } from '@functions';
import { Category, Product } from '@models';

export default async (req, res) => {
  const { slug } = req.body;

  const alreadyExistsSlug = await Product.findOne({ slug });
  if (alreadyExistsSlug) {
    throw error(400, 'A product with this slug already exists');
  }

  // Validate related document exists
  const categoryExists = await Category.findById(req.body?.category?._id);
  if (!categoryExists) {
    throw error(400, 'Category not found');
  }

  const document = await Product.create(req.body);

  return res.status(200).json({
    data: document,
    message: 'Product created successfully!',
  });
};
```

## File 7: `controllers/product/update.js` (validates related docs, updates trails)

```js
import { error, updateProductVariants } from '@functions';
import { Category, Product } from '@models';

export default async (req, res) => {
  const { id } = req.params;
  const { slug } = req.body;

  if (slug) {
    const existingBySlug = await Product.findOne({ slug, _id: { $ne: id } });
    if (existingBySlug) {
      throw error(400, 'A product with this slug already exists');
    }
  }

  const categoryExists = await Category.findById(req.body?.category?._id);
  if (!categoryExists) {
    throw error(400, 'Category not found');
  }

  const document = await Product.findByIdAndUpdate(id, req.body, { new: true });
  if (!document) {
    throw error(404, 'Product not found');
  }

  // If this resource is embedded as a ref in child models, update their trails
  await updateProductVariants(document);

  return res.status(200).json({
    data: document,
    message: 'Product updated successfully!',
  });
};
```
