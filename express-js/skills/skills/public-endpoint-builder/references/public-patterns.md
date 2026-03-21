# Public Controller Patterns

Real examples from the codebase showing different public endpoint patterns.

## Pattern 1: Simple List — `get-products.js`

Paginated list with aggregation pipeline and related data via `$lookup`.

```js
import { applyFiltersProduct } from '@filters';
import { Product } from '@models';

export default async (req, res) => {
  const page = parseInt(req.query.page, 10) || 1;
  const perPage = parseInt(req.query.perPage, 10) || 10;

  const productFilters = {
    ...applyFiltersProduct(req.query),
    isActive: true,
  };

  const totalCount = await Product.countDocuments(productFilters);

  const documents = await Product.aggregate([
    { $match: productFilters },
    {
      $lookup: {
        from: 'product_variants',
        localField: '_id',
        foreignField: 'product._id',
        as: 'variants',
      },
    },
    {
      $addFields: {
        defaultVariant: {
          $let: {
            vars: {
              defaultActive: {
                $filter: {
                  input: '$variants',
                  cond: {
                    $and: [
                      { $eq: ['$$this.isDefault', true] },
                      { $eq: ['$$this.isActive', true] },
                    ],
                  },
                },
              },
              anyActive: {
                $filter: {
                  input: '$variants',
                  cond: { $eq: ['$$this.isActive', true] },
                },
              },
            },
            in: { $ifNull: [{ $first: '$$defaultActive' }, { $first: '$$anyActive' }] },
          },
        },
      },
    },
    { $match: { defaultVariant: { $ne: null } } },
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

## Pattern 2: Detail Page — `view-page-product.js`

Single resource by slug with multiple `$lookup` stages to populate related data.

```js
import { error } from '@functions';
import { Product } from '@models';

export default async (req, res) => {
  const { slug } = req.params;

  if (!slug) {
    throw error(400, 'Slug is required');
  }

  const product = await Product.aggregate([
    { $match: { slug } },
    // Populate productType
    {
      $lookup: {
        from: 'product_types',
        localField: 'productType._id',
        foreignField: '_id',
        as: 'productTypeData',
      },
    },
    {
      $addFields: {
        productType: {
          $mergeObjects: ['$productType', { $arrayElemAt: ['$productTypeData', 0] }],
        },
      },
    },
    // Populate category
    {
      $lookup: {
        from: 'categories',
        localField: 'category._id',
        foreignField: '_id',
        as: 'categoryData',
      },
    },
    {
      $addFields: {
        category: {
          $mergeObjects: ['$category', { $arrayElemAt: ['$categoryData', 0] }],
        },
      },
    },
    // Clean up temp fields
    {
      $project: {
        productTypeData: 0,
        categoryData: 0,
      },
    },
  ]);

  if (product.length === 0) {
    throw error(404, 'Product not found');
  }

  return res.status(200).json({ product: product[0] });
};
```

## Pattern 3: Simple Find (no aggregation)

For lightweight endpoints that don't need populated data.

```js
import { Resource } from '@models';

export default async (req, res) => {
  const documents = await Resource.find({ isActive: true })
    .select('name slug')
    .sort({ name: 1 })
    .lean();

  return res.status(200).json(documents);
};
```

## Route Registration Pattern

In `routes/public.js`:

```js
import { Public } from '@controllers';
import { diacriticInsensitive } from '@middleware';
import { Router } from 'express';

const router = Router();
export default router;

// Products
router.get('/public/get-products', diacriticInsensitive(['search']), Public.getProducts);
router.get('/public/view-page-products', diacriticInsensitive(['search']), Public.viewPageProducts);
router.get('/public/view-page-product/:slug', Public.viewPageProduct);

// Articles
router.get('/public/articles', diacriticInsensitive(['search']), Public.viewPageArticles);
router.get('/public/articles/:slug', Public.viewPageArticle);
```

## Response Shapes

**List endpoints** return paginated response:
```json
{
  "pageParams": { "count": 50, "hasNext": true, "page": 1, "perPage": 10 },
  "pages": [...]
}
```

**Detail endpoints** return the document (optionally wrapped):
```json
{ "product": { "_id": "...", "title": "...", ... } }
```
Or directly:
```json
{ "_id": "...", "title": "...", ... }
```

## MongoDB Collection Names for `$lookup`

The `from` field in `$lookup` uses the MongoDB collection name, which is the **lowercase plural** of the model name, with spaces/camelCase converted to underscores:
- `product` → `products`
- `productVariant` → `product_variants`
- `productType` → `product_types`
- `category` → `categories`
- `gallery` → `galleries`
