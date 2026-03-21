# Script Template

Complete example based on the import-articles script.

## Script File — `scripts/import-articles.js`

```js
import Article from '@models/article';
import { runScript } from 'express-goodies/functions';
import blogArticles from '../constants/blog-articles';

/**
 * Usage:
 * node npm.js import-articles
 */
runScript(script);

async function script() {
  console.log('Starting blog articles import...');

  let created = 0;
  let skipped = 0;

  for (const articleData of blogArticles) {
    const existing = await Article.findOne({ slug: articleData.slug });

    if (existing) {
      console.log(`Skipped (already exists): ${articleData.slug}`);
      skipped++;
      continue;
    }

    await Article.create(articleData);
    console.log(`Created: ${articleData.title}`);
    created++;
  }

  console.log(`\nImport complete: ${created} created, ${skipped} skipped`);
}
```

### Key patterns:

1. **`runScript(fn)`** — from `express-goodies/functions`. Handles:
   - Connecting to MongoDB (using `MONGODB_URI` from env)
   - Running the script function
   - Disconnecting
   - Exiting the process

2. **Usage comment** — document how to run the script: `node npm.js {script-name}`

3. **`runScript(script)` before function definition** — hoisting makes this work. The call must be at module level.

4. **Idempotent logic** — check if data exists before creating to make the script safe to re-run.

5. **Progress logging** — log each operation and a final summary.

## Running Scripts

Scripts are run via the custom npm runner:

```bash
node npm.js {script-name}
```

Where `{script-name}` matches the filename without `.js`. For example:
- `node npm.js import-articles` → runs `scripts/import-articles.js`
- `node npm.js reset-login-attempts` → runs `scripts/reset-login-attempts.js`

## Other Script Examples

### Bulk Update — `scripts/reset-login-attempts.js`

```js
import { Identity } from '@models';
import { runScript } from 'express-goodies/functions';

/**
 * Usage:
 * node npm.js reset-login-attempts
 */
runScript(script);

async function script() {
  console.log('Resetting login attempts...');

  const result = await Identity.updateMany(
    { loginAttempts: { $gt: 0 } },
    { $set: { loginAttempts: 0, lockUntil: null } }
  );

  console.log(`Reset ${result.modifiedCount} accounts`);
}
```

### Data Migration Pattern

```js
import { Resource } from '@models';
import { runScript } from 'express-goodies/functions';

/**
 * Usage:
 * node npm.js migrate-resource-slugs
 */
runScript(script);

async function script() {
  console.log('Migrating slugs...');

  const documents = await Resource.find({ slug: { $exists: false } });

  for (const doc of documents) {
    const slug = doc.name
      .toLowerCase()
      .replace(/[^a-z0-9]+/g, '-')
      .replace(/^-|-$/g, '');

    await Resource.findByIdAndUpdate(doc._id, { slug });
    console.log(`${doc.name} → ${slug}`);
  }

  console.log(`\nMigrated ${documents.length} documents`);
}
```
