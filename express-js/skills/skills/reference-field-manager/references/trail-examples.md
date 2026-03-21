# Trail Function Examples

Trail functions propagate changes when a referenced document is updated or deleted.

## Update Trails

When a referenced document's fields change, all documents that embed a copy of those fields need to be updated.

### Example: `functions/update-tag-trails.js`

Tag is stored as an array ref `tags: [{ _id, name }]` in Gallery.

```js
import { Gallery } from '../models';

const updateTagTrails = async (tag) => {
  for (const Model of [Gallery]) {
    await Model.updateMany(
      { 'tags._id': tag?._id },
      {
        'tags.$.name': tag?.name,
      }
    );
  }
};

export default updateTagTrails;
```

**Key patterns:**
- `'tags._id': tag?._id` — finds documents where the array contains an item with matching `_id`
- `'tags.$.name'` — the `$` positional operator updates the matched array element
- Use optional chaining (`?.`) on the input parameter

### Example: `functions/update-category-trails.js`

Category is stored as a single ref `category: { _id, name, slug }` in Product.

```js
import { Product } from '../models';

const updateCategoryTrails = async (category) => {
  for (const Model of [Product]) {
    await Model.updateMany(
      { 'category._id': category?._id },
      {
        'category.name': category?.name,
        'category.slug': category?.slug,
      }
    );
  }
};

export default updateCategoryTrails;
```

**Key difference:** No `$` positional operator for single refs — just direct field paths.

### Multiple referencing models

If a resource is embedded in multiple models, add them all to the array:

```js
for (const Model of [Product, Article, Portfolio]) {
  await Model.updateMany(...);
}
```

## Remove Trails

When a referenced document is deleted, clean up the embedded copies.

### Example: `functions/remove-tag-trails.js` (array ref)

```js
import { Gallery } from '../models';

const removeTagTrails = async (tagId) => {
  for (const Model of [Gallery]) {
    await Model.updateMany(
      { 'tags._id': tagId },
      { $pull: { tags: { _id: tagId } } }
    );
  }
};

export default removeTagTrails;
```

**Key pattern:** `$pull` removes the matching element from the array.

### For single refs (not arrays)

Set the field to `null` instead of using `$pull`:

```js
const removeAuthorTrails = async (authorId) => {
  for (const Model of [Article]) {
    await Model.updateMany(
      { 'author._id': authorId },
      { author: null }
    );
  }
};
```

## Wiring Into Controllers

### In `controllers/{referenced}/update.js`

```js
import { error, update{Referenced}Trails } from '@functions';
import { {Referenced} } from '@models';

export default async (req, res) => {
  const { id } = req.params;
  // ... duplicate check ...

  const updated = await {Referenced}.findByIdAndUpdate(id, req.body, { new: true });
  await update{Referenced}Trails(updated);

  return res.status(200).json({
    data: updated,
    message: '{Referenced} updated successfully!',
  });
};
```

### In `controllers/{referenced}/delete.js`

```js
import { remove{Referenced}Trails } from '@functions';
import { {Referenced} } from '@models';

export default async (req, res) => {
  const { id } = req.params;

  await {Referenced}.findByIdAndDelete(id);
  await remove{Referenced}Trails(id);

  return res.status(200).json({
    data: req.model,
    message: '{Referenced} deleted successfully!',
  });
};
```

## Naming Convention

- Update function: `update{Resource}Trails` (e.g. `updateTagTrails`, `updateCategoryTrails`)
- Remove function: `remove{Resource}Trails` (e.g. `removeTagTrails`, `removeCategoryTrails`)
- File names: `update-{resource}-trails.js`, `remove-{resource}-trails.js`

## Important

- Use relative imports (`'../models'`) inside `functions/` since there's no `@` alias for the functions directory itself
- Use optional chaining (`?.`) when accessing properties of the input parameter
- The `for...of` loop pattern allows easy addition of more models later
