# Public Endpoint Naming Guide

## Two Naming Patterns

### `get-{resources}` — Lightweight Data Fetching

Use for:
- Dropdown/selector data (e.g. list of categories for a filter dropdown)
- Simple lists with minimal fields
- API-style data endpoints that return raw data

Examples:
- `get-products` — product list for a catalog
- `get-product-types` — product type list for a filter
- `get-shops` — shop locations list

Characteristics:
- Usually returns a flat array or simple paginated list
- Minimal data per item (just what's needed for display)
- May use `.select()` to limit fields
- Often no pagination needed (small datasets)

### `view-page-{resource(s)}` — Full Page Data

Use for:
- Server-side page data that powers an entire frontend page
- Complex data with populated relations
- Detail pages (by slug)
- List pages with rich data per item

**Plural** (`view-page-{resources}`) for list pages:
- `view-page-products` — products listing page
- `view-page-articles` — blog listing page
- `view-page-bestsellers` — bestsellers page

**Singular** (`view-page-{resource}`) for detail pages:
- `view-page-product/:slug` — single product page
- `view-page-article/:slug` — single article page

Characteristics:
- Returns all data needed to render a full page
- Often uses aggregation pipelines with `$lookup`
- Includes related/populated data
- Detail pages look up by slug (not ID)

## Route URL Patterns

```
/public/get-{resources}                    → lightweight list
/public/view-page-{resources}              → full list page
/public/view-page-{resource}/:slug         → detail page by slug
```

## File Name = Export Name Mapping

| File | Export Name |
|------|------------|
| `get-products.js` | `getProducts` |
| `view-page-product.js` | `viewPageProduct` |
| `view-page-products.js` | `viewPageProducts` |
| `view-page-homepage.js` | `viewPageHomepage` |

Convert kebab-case filename to camelCase for the export name.
