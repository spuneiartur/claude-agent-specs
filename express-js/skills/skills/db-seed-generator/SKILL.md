---
name: db-seed-generator
description: Generate database seed files and one-off import/migration scripts for an Express.js + MongoDB API. Creates numbered seed runners in db/seeds/ and standalone scripts in scripts/. Use this skill whenever the user wants to create seed data, import data from CSV or JSON, write a migration script, populate the database, do a bulk update, or says things like "seed the database with test data" or "import products from this JSON" or "write a script to reset all user passwords".
---

# Database Seed & Script Generator

Generates database seed files (`db/seeds/`) for repeatable initial data and standalone scripts (`scripts/`) for one-off migrations and imports. Both follow established patterns and connect to MongoDB properly.

## When to Use

- "Create seed data for reviews"
- "Import products from this JSON file"
- "Write a migration script to add slugs to all categories"
- "Seed the database with test users"
- "Bulk update all products to set isActive = true"

## Two Types

### Seeds (`db/seeds/`)
- **Repeatable** initial data loaded via `npm run seed`
- Numbered for execution order: `001_identities.js`, `002_categories.js`
- Data factories live in `db/resources/`
- Registered in `db/seeds/index.js`

### Scripts (`scripts/`)
- **One-off** operations: imports, migrations, data fixes
- Run via `node npm.js {script-name}` (without `.js` extension)
- Use `runScript()` from `express-goodies/functions` for DB connection
- Self-contained — handle their own connection and logging

## Workflow

### For Seeds

#### Step 1: Determine sequence number

Check existing seeds in `db/seeds/` to find the next number:
```
001_identities.js
002_xxx.js  ← next would be 002 or whatever follows
```

#### Step 2: Create the data factory — `db/resources/{resource}.js`

Read `references/seed-template.md` for the complete pattern.

```js
const {resources} = () => {
  return [
    {
      name: 'Example Item 1',
      // fields matching the model schema
    },
    {
      name: 'Example Item 2',
    },
  ];
};

export default {resources};
```

If data needs async processing (e.g. hashing passwords), make it async:

```js
import bcrypt from 'bcryptjs';

const identities = async () => {
  const password = await bcrypt.hash('password123', 10);
  return [
    { email: 'admin@example.com', password, role: 'admin' },
  ];
};

export default identities;
```

#### Step 3: Create the seed runner — `db/seeds/{NNN}_{resource}.js`

```js
import { {Resource} } from '@models';
import {resources} from '../resources/{resources}';

export async function seed() {
  try {
    console.log('Planting seeds for {resources}...');

    const seeds = await {resources}();
    await {Resource}.insertMany(seeds);

    console.log('✓');
  } catch (err) {
    console.warn('Error! Cannot insert {resources}');
    console.error(err);
  }
}
```

#### Step 4: Register in `db/seeds/index.js`

```js
export * as {resources} from './{NNN}_{resources}';
```

### For Scripts

#### Step 1: Create the script — `scripts/{action}-{resource}.js`

Read `references/script-template.md` for the complete pattern.

```js
import { {Resource} } from '@models';
import { runScript } from 'express-goodies/functions';

/**
 * Usage:
 * node npm.js {action}-{resource}
 */
runScript(script);

async function script() {
  console.log('Starting {action} for {resources}...');

  // Script logic here

  console.log('Done!');
}
```

**`runScript()`** handles: connecting to MongoDB, running the script function, disconnecting, and exiting the process. You don't need to manage the DB connection manually.

#### Common Script Patterns

**Import from JSON/array:**
```js
async function script() {
  console.log('Starting import...');

  let created = 0;
  let skipped = 0;

  for (const item of data) {
    const existing = await Resource.findOne({ slug: item.slug });

    if (existing) {
      console.log(`Skipped (already exists): ${item.slug}`);
      skipped++;
      continue;
    }

    await Resource.create(item);
    console.log(`Created: ${item.name}`);
    created++;
  }

  console.log(`\nImport complete: ${created} created, ${skipped} skipped`);
}
```

**Bulk update:**
```js
async function script() {
  console.log('Updating all resources...');

  const result = await Resource.updateMany(
    { isActive: { $exists: false } },
    { $set: { isActive: true } }
  );

  console.log(`Updated ${result.modifiedCount} documents`);
}
```

**Migration (add field/transform data):**
```js
async function script() {
  console.log('Migrating resources...');

  const documents = await Resource.find({});

  for (const doc of documents) {
    const slug = doc.name.toLowerCase().replace(/\s+/g, '-');
    await Resource.findByIdAndUpdate(doc._id, { slug });
    console.log(`Migrated: ${doc.name} → ${slug}`);
  }

  console.log(`Migrated ${documents.length} documents`);
}
```

## Important Rules

- Seeds use `insertMany` for bulk inserts — fast but no validation hooks
- Scripts use `create` or `findByIdAndUpdate` for individual operations — slower but triggers hooks
- Always add console logging for progress tracking
- Scripts should be idempotent when possible (check for existing data before creating)
- Use `@models` alias for model imports
- Use `runScript()` from `express-goodies/functions` for scripts — it handles DB connection

## Reference Files

- `references/seed-template.md` — Complete seed example (identities)
- `references/script-template.md` — Complete script example (import-articles)
