---
name: admin-crud-scaffold
description: >
  Scaffold a complete admin CRUD feature with all required files: API service, validation models,
  table columns, admin pages (list/add/edit), admin components (header, table, filters, actions cell),
  and form components (shared form, add form, edit form). Use this skill whenever the user wants to
  add a new admin section, create CRUD pages, scaffold entity management, add an admin table, build
  admin forms for a new resource, or mentions "scaffold", "generate", "create admin pages for",
  "add management for", or "CRUD for [entity]". This is the most common development pattern in the
  project — trigger it proactively when the user describes any new entity that needs admin management.
---

# Admin CRUD Scaffold

Generate the complete file set for a new admin CRUD entity. This pattern repeats across every managed entity in the project (articles, categories, products, tags, textures, portfolios, etc.) and requires ~15 coordinated files across 6 directories.

## Before You Start

Ask the user for:
1. **Entity name** (singular, e.g., "testimonial", "order", "service")
2. **Fields** — what data does this entity have? (name, type, required?)
3. **Filter fields** — which fields should be filterable in the admin list? (search, status, dates?)
4. **Table columns** — which fields should appear in the admin table?

From the entity name, derive:
- `{entity}` — lowercase singular (e.g., `testimonial`)
- `{Entity}` — PascalCase singular (e.g., `Testimonial`)
- `{entities}` — lowercase plural (e.g., `testimonials`)
- `{Entities}` — PascalCase plural (e.g., `Testimonials`)

## File Generation Checklist

Generate all files in order. Read `references/patterns.md` for exact code templates.

### 1. API Service — `api/{entity}.js`

```js
import { axiosAuth } from '@lib';

export const get{Entity} = (id) => axiosAuth.get(`/admin/{entities}/${id}`);
export const create{Entity} = (data) => axiosAuth.post('/admin/{entities}', data);
export const update{Entity} = ({ id, data }) => axiosAuth.put(`/admin/{entities}/${id}`, data);
export const delete{Entity} = (id) => axiosAuth.delete(`/admin/{entities}/${id}`);
```

### 2. Validation Model — `models/{entity}.js`

Export `validationSchema` (Yup.object().shape({...})) and `initialValues` (plain object). Use Yup validators matching each field's type and requirements.

### 3. Filter Model — `models/{entity}-filters.js`

Export `validationSchema` and `initialValues` for filter fields (typically `search`, `status`, date ranges). All filter fields are optional strings.

### 4. Table Columns — `data/{entity}-columns.js`

Array of column objects with `Header`, `accessor`, `Cell`, `extraClass`, `disableSortBy`. The last column must be the actions cell with `accessor: '_id'` and `Cell: {Entity}ActionsCell`. Update the barrel export in `data/index.js`.

### 5. Admin List Page — `pages/admin/{entities}/index.js`

```jsx
import { checkAuth, withAuth } from '@auth';
import { Layout } from '@components';
import { {Entity}Header, {Entity}Table } from '@components/Admin/{Entity}';
import { useState } from 'react';

const Page = () => {
  const [options, setOptions] = useState({});
  return (
    <Layout title="{Entities}">
      <{Entity}Header setOptions={setOptions} />
      <{Entity}Table options={options} />
    </Layout>
  );
};

export async function getServerSideProps(context) {
  return await checkAuth(context);
}
export default withAuth(Page);
```

### 6. Admin Add Page — `pages/admin/{entities}/add/index.js`

```jsx
import { checkAuth, withAuth } from '@auth';
import { Layout } from '@components';
import { Add{Entity}Form } from '@components/Forms';

const Page = () => (
  <Layout title="Add {Entity}">
    <Add{Entity}Form />
  </Layout>
);

export async function getServerSideProps(context) {
  return await checkAuth(context);
}
export default withAuth(Page);
```

### 7. Admin Edit Page — `pages/admin/{entities}/edit/[id].js`

Uses `useQuery` to fetch the entity by `router.query.id`. Renders loading/error/success states. Passes the entity data to `Edit{Entity}Form`.

```jsx
import { checkAuth, withAuth } from '@auth';
import { Button, Layout } from '@components';
import { Edit{Entity}Form } from '@components/Forms';
import { useQuery } from '@hooks';
import { useRouter } from 'next/router';

const Page = () => {
  const router = useRouter();
  const { id } = router.query;
  const { data: {entity}, status } = useQuery(`admin/{entities}/${id}`);

  return (
    <Layout title="Edit {Entity}">
      {status === 'loading' && <div>Loading...</div>}
      {status === 'error' && <div>Error loading {entity}.</div>}
      {status === 'success' && <Edit{Entity}Form {entity}={{entity}} />}
    </Layout>
  );
};

export async function getServerSideProps(context) {
  return await checkAuth(context);
}
export default withAuth(Page);
```

### 8. Admin Components — `components/Admin/{Entity}/`

Create a directory with these files:

**{Entity}Header.jsx** — Title, add button (links to add page), filters section.
```jsx
import { Button } from '@components';
import { {Entity}Filters } from '@components/Admin/{Entity}';

const {Entity}Header = ({ setOptions }) => (
  <div className="bg-white rounded-xl shadow-sm border border-metal-200 p-6 mb-6">
    <div className="flex flex-col sm:flex-row sm:items-center sm:justify-between gap-4 mb-6">
      <div>
        <h1 className="text-2xl font-bold text-gray-900">{Entities}</h1>
        <p className="text-gray-600">Manage {entities}</p>
      </div>
      <Button href="/admin/{entities}/add" className="px-4 py-2.5 bg-gradient-to-r from-accent to-accent/90 text-white rounded-lg hover:from-accent/90 hover:to-accent/80 transition-all duration-200 flex items-center space-x-2 shadow-sm hover:shadow-md font-medium">
        <i className="fas fa-plus text-sm"></i>
        <span>Add {Entity}</span>
      </Button>
    </div>
    <div className="pt-4 border-t border-gray-100">
      <{Entity}Filters setOptions={setOptions} />
    </div>
  </div>
);
export default {Entity}Header;
```

**{Entity}Table.jsx** — Uses `useInfiniteQuery` with table components.
```jsx
import { LoadMoreOnClick } from '@components/Buttons';
import { TableColumns, TableError, TableLoading, TableSuccess } from '@components/Tables';
import { {entity}Columns } from '@data';
import { useInfiniteQuery } from '@hooks';

const {Entity}Table = ({ options }) => {
  const { data, status, ...props } = useInfiniteQuery('admin/{entities}', options);
  return (
    <>
      <TableColumns pageParams={data?.pageParams} />
      {status === 'loading' && <TableLoading name="{entities}" columns={{entity}Columns} />}
      {status === 'error' && <TableError name="{entities}" columns={{entity}Columns} />}
      {status === 'success' && (
        <>
          <TableSuccess name="{entities}" columns={{entity}Columns} data={data} {...props} />
          <div className="px-4 sm:p-4"><LoadMoreOnClick {...props} /></div>
        </>
      )}
    </>
  );
};
export default {Entity}Table;
```

**{Entity}Filters.jsx** — Uses `AutoSubmitFilterForm` with filter model.
```jsx
import { Search, Dropdown, DatePicker } from '@components/Fields';
import { Field } from '@components/HookForm';
import AutoSubmitFilterForm from '@components/HookForm/AutoSubmitFilterForm';
import { initialValues, validationSchema } from '@models/{entity}-filters';

const {Entity}Filters = ({ setOptions }) => (
  <AutoSubmitFilterForm initialValues={initialValues} validationSchema={validationSchema} onSubmit={setOptions}>
    <div className="grid grid-cols-1 gap-4 md:grid-cols-2 lg:grid-cols-3">
      <Field as={Search} name="search" placeholder="Search {entities}..." label="Search" />
      {/* Add filter fields based on entity requirements */}
    </div>
  </AutoSubmitFilterForm>
);
export default {Entity}Filters;
```

**{Entity}ActionsCell.jsx** — Edit link + delete with confirmation.
```jsx
import { delete{Entity} } from '@api/{entity}';
import { AreYouSure, Button } from '@components';
import { useDisclosure, useMutation } from '@hooks';

const {Entity}ActionsCell = ({ value: id, row: { original } }) => {
  const { isOpen, show, hide } = useDisclosure();
  const mutation = useMutation(delete{Entity}, { invalidateQueries: 'admin/{entities}' });
  const handleDelete = async () => { await mutation.mutateAsync(id); hide(); };

  return (
    <div className="flex items-center justify-center space-x-2">
      <Button href={`/admin/{entities}/edit/${id}`} className="text-primary border-primary hover:bg-primary hover:text-white w-8 h-8 p-0 flex items-center justify-center rounded-md" title="Edit">
        <i className="fas fa-edit text-xs"></i>
      </Button>
      <Button onClick={show} className="text-red-600 border-red-600 hover:bg-red-600 hover:text-white w-8 h-8 p-0 flex items-center justify-center rounded-md" title="Delete">
        <i className="fas fa-trash text-xs"></i>
      </Button>
      <AreYouSure isOpen={isOpen} hide={hide} onConfirm={handleDelete} title="Confirm deletion" message={`Are you sure you want to delete "${original.title || original.name}"?`} isLoading={mutation.isPending} />
    </div>
  );
};
export default {Entity}ActionsCell;
```

**index.js** — Barrel exports for all components in the directory.

### 9. Form Components — `components/Forms/`

**{Entity}Form.jsx** — Shared form fields using `Field` with appropriate field components from `@components/Fields`. Uses `useFormContext` if needed for conditional logic.

**Add{Entity}Form.jsx** — Wraps `{Entity}Form` with `HookForm`/`Form`/`Submit`. Uses `useMutation(create{Entity}, { invalidateQueries: 'admin/{entities}', successCallback: () => router.push('/admin/{entities}') })`.

**Edit{Entity}Form.jsx** — Same structure, receives `{entity}` as prop, uses `initialValues={{entity}}` and `useMutation(update{Entity})` passing `{ id: {entity}._id, data }`.

### 10. Barrel Export Updates

Add exports to:
- `data/index.js`: `export { default as {entity}Columns } from './{entity}-columns';`
- `components/Forms/index.js`: `export { default as Add{Entity}Form } from './Add{Entity}Form';` and `export { default as Edit{Entity}Form } from './Edit{Entity}Form';`

## Important Conventions

- **Max 40-50 lines per file** — keep each file short. If a component grows beyond this, extract sub-components.
- **Separate status components** — create `{Entity}FormLoading.jsx` and `{Entity}FormError.jsx` in `components/Admin/{Entity}/` using `Bone` component for loading skeletons. The edit page uses these for the `useQuery` loading/error states.
- **Font Awesome icons** — use `<i className="fas fa-plus"></i>` format.
- **No arbitrary Tailwind values** — don't use `p-[50px]` syntax.
- **Check `examples/` folder first** — look for similar implementations before generating.

## After Generation

1. Verify all imports resolve correctly
2. Check that barrel exports are updated
3. Remind the user to add a navigation link in the admin sidebar/menu
4. Remind the user that the backend API endpoints (`/admin/{entities}`) must exist
