# Seed Template

Complete example based on the identities seed.

## Seed Runner — `db/seeds/001_identities.js`

```js
import { Identity } from '@models';
import identities from '../resources/identities';

export async function seed() {
  try {
    console.log('Planting seeds for identities...');

    const seeds = await identities();
    await Identity.insertMany(seeds);

    console.log('✓');
  } catch (err) {
    console.warn('Error! Cannot insert identities');
    console.error(err);
  }
}
```

### Key patterns:
- Named export `seed` (not default export)
- Import model from `@models`
- Import data factory from `../resources/{resource}`
- `await` the factory (it may be async, e.g. for password hashing)
- Use `insertMany` for bulk insert
- Wrap in try/catch for seed-level error handling (seeds ARE the exception to the no-try/catch rule)
- Console log progress with `✓` on success

## Data Factory — `db/resources/identities.js`

```js
import bcrypt from 'bcryptjs';

const identities = async () => {
  const password = await bcrypt.hash('test1234', 10);

  return [
    {
      email: 'admin@example.com',
      password,
      firstName: 'Admin',
      lastName: 'User',
      role: 'admin',
    },
    {
      email: 'user@example.com',
      password,
      firstName: 'Test',
      lastName: 'User',
      role: 'user',
    },
  ];
};

export default identities;
```

### Key patterns:
- Default export — a function (sync or async) that returns an array
- Use async when data needs processing (hashing, date generation, etc.)
- Return plain objects matching the model schema

## Seed Registry — `db/seeds/index.js`

```js
export * as identities from './001_identities';
export * as categories from './002_categories';
```

- Namespace exports (`export * as {name}`)
- Ordered by sequence number
- The seed runner (`npm run seed`) imports and calls each `seed()` function

## Running Seeds

```bash
npm run seed
```

This calls `runSeeds()` from `express-goodies` which:
1. Connects to MongoDB
2. Imports all seed modules from `db/seeds/index.js`
3. Calls each `seed()` function in order
4. Disconnects
