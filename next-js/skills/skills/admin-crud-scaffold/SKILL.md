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

## Verify the project first — 60 seconds, saves an hour

This skill targets a family of projects that drift apart. **Check these four things before generating**, and adapt the templates if the project differs:

| Check | Command | Templates assume |
|---|---|---|
| React Query major version | `grep '@tanstack/react-query' package.json` | **v5** → status is `'pending'` (v4 uses `'loading'`) |
| Table library version | `grep '@tanstack/react-table' package.json` | **v8** → columns use `id`/`header`/`accessorKey`/`cell` (v6/v7 use `Header`/`accessor`/`Cell`) |
| Admin URL prefix | `grep mgt-portal next.config.js` | If the rewrite exists, all admin `href`s must be **`/mgt-portal/admin/...`** — `/admin/*` redirects to `/404`. If absent, use `/admin/...` |
| Icon set | `grep -E 'lucide-react\|font-awesome' package.json` | **Lucide** — `import { Plus } from 'lucide-react'` then `<Plus className="size-4" />` |

Also confirm the components you are about to import actually exist. The barrels are the source of truth: `components/index.js`, `components/Fields/index.js`, `components/HookForm/index.js`, `components/Tables/index.js`. Don't import a name because another project has it.

## File Generation Checklist

Generate all files in order. Read `references/patterns.md` for exact code templates.

### 1. API Service — `api/{entity}.js`

Exports ordered A-Z (project convention).

```js
import { axiosAuth } from '@lib';

export const create{Entity} = (data) => axiosAuth.post('/admin/{entities}', data);
export const delete{Entity} = (id) => axiosAuth.delete(`/admin/{entities}/${id}`);
export const get{Entity} = (id) => axiosAuth.get(`/admin/{entities}/${id}`);
export const update{Entity} = ({ id, data }) => axiosAuth.put(`/admin/{entities}/${id}`, data);
```

### 2. Validation Model — `models/{entity}.js`

Export `validationSchema` (Yup.object().shape({...})) and `initialValues` (plain object). Use Yup validators matching each field's type and requirements.

### 3. Filter Model — `models/{entity}-filters.js`

Export `validationSchema` and `initialValues` for filter fields (typically `search`, `status`, date ranges). All filter fields are optional strings.

### 4. Table Columns — `data/{entity}-columns.js`

**TanStack Table v8 format.** Each column is `{ id, header, accessorKey, cell?, extraClass? }`:

- `id` — unique string, required (used as React key by `TableHeader`)
- `header` — the visible label (string or component)
- `accessorKey` — path into the row object; supports dot notation (`'identity.name'`)
- `cell` — optional component; receives the TanStack cell context, **not** a `value` prop
- `extraClass` — non-standard, read by `TableHeader.jsx` for column widths

The last column is the actions cell. Update the barrel in `data/index.js`.

```js
import { Time } from '@components';
import { {Entity}ActionsCell, {Entity}StatusCell } from '@components/Admin/{Entity}';

const {entity}Columns = [
  {
    id: 'name',
    header: 'Nume',
    accessorKey: 'name',
    extraClass: 'font-medium text-gray-900',
  },
  {
    id: 'status',
    header: 'Status',
    accessorKey: 'status',
    cell: {Entity}StatusCell,
    extraClass: 'w-24 text-center',
  },
  {
    id: 'createdAt',
    header: 'Creat la',
    accessorKey: 'createdAt',
    cell: Time,
    extraClass: 'w-32',
  },
  {
    id: 'actions',
    header: 'Acțiuni',
    accessorKey: '_id',
    cell: {Entity}ActionsCell,
    extraClass: 'w-32 text-center',
  },
];

export default {entity}Columns;
```

> ⚠️ Do not use the v6 shape (`Header`, `accessor`, `Cell`). `TableSuccess.jsx` passes columns straight to `useReactTable`, and `TableHeader`/`TableRow` call `flexRender(column.columnDef.header)` / `flexRender(cell.column.columnDef.cell)`. v6 keys are silently ignored and you get an empty table with no error.

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
  <Layout title="Adaugă {Entity}">
    <Add{Entity}Form />
  </Layout>
);

export async function getServerSideProps(context) {
  return await checkAuth(context);
}
export default withAuth(Page);
```

### 7. Admin Edit Page — `pages/admin/{entities}/edit/[id].js`

Fetches the entity by `router.query.id` and renders the three query states with the dedicated status components.

```jsx
import { checkAuth, withAuth } from '@auth';
import { Layout } from '@components';
import { {Entity}FormError, {Entity}FormLoading } from '@components/Admin/{Entity}';
import { Edit{Entity}Form } from '@components/Forms';
import { useQuery } from '@hooks';
import { useRouter } from 'next/router';

const Page = () => {
  const router = useRouter();
  const { id } = router.query;
  const { data: {entity}, status } = useQuery(
    `admin/{entities}/${id}`,
    {},
    { enabled: Boolean(id) }
  );

  return (
    <Layout title="Editează {Entity}">
      {status === 'pending' && <{Entity}FormLoading />}
      {status === 'error' && <{Entity}FormError message="{Entity} negăsit!" />}
      {status === 'success' && <Edit{Entity}Form {entity}={{entity}} />}
    </Layout>
  );
};

export async function getServerSideProps(context) {
  return await checkAuth(context);
}
export default withAuth(Page);
```

> `enabled: Boolean(id)` matters. On the first render of a Pages Router dynamic route, `router.query` is empty, so without the guard the hook fires `GET /admin/{entities}/undefined` and flashes the error state.

### 8. Admin Components — `components/Admin/{Entity}/`

Create a directory with these files:

**{Entity}Header.jsx** — Title, add button, filters section.
```jsx
import { Button } from '@components';
import { {Entity}Filters } from '@components/Admin/{Entity}';
import { Plus } from 'lucide-react';

const {Entity}Header = ({ setOptions }) => (
  <div className="mb-6 rounded-xl border border-gray-200 bg-white p-6 shadow-sm">
    <div className="mb-6 flex flex-col gap-4 sm:flex-row sm:items-center sm:justify-between">
      <div>
        <h1 className="text-2xl font-bold text-gray-900">{Entities}</h1>
        <p className="text-gray-600">Gestionează {entities}</p>
      </div>
      <Button
        href="/mgt-portal/admin/{entities}/add"
        className="flex items-center gap-2 rounded-lg bg-primary px-4 py-2.5 font-medium text-white transition-colors hover:bg-primary/90"
      >
        <Plus className="size-4" />
        <span>Adaugă {Entity}</span>
      </Button>
    </div>
    <div className="border-t border-gray-100 pt-4">
      <{Entity}Filters setOptions={setOptions} />
    </div>
  </div>
);

export default {Entity}Header;
```

**{Entity}Table.jsx** — `useInfiniteQuery` + the table components.
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

      {status === 'pending' && <TableLoading name="{entities}" columns={{entity}Columns} />}
      {status === 'error' && <TableError name="{entities}" columns={{entity}Columns} />}
      {status === 'success' && (
        <>
          <TableSuccess name="{entities}" columns={{entity}Columns} data={data} />
          <div className="px-4 sm:p-4">
            <LoadMoreOnClick {...props} />
          </div>
        </>
      )}
    </>
  );
};

export default {Entity}Table;
```

> `useInfiniteQuery` in this project already flattens pages in its `select`: `data.pages` is a flat array of documents, and `data.pageParams` is the last page's meta. Do not flatten again.

**{Entity}Filters.jsx** — `AutoSubmitForm` (debounced, no submit button) with the filter model.
```jsx
import { DatePicker, Dropdown, Search } from '@components/Fields';
import { AutoSubmitForm, Field } from '@components/HookForm';
import { initialValues, validationSchema } from '@models/{entity}-filters';

const {Entity}Filters = ({ setOptions }) => (
  <AutoSubmitForm
    initialValues={initialValues}
    validationSchema={validationSchema}
    onSubmit={setOptions}
  >
    <div className="grid grid-cols-1 gap-4 md:grid-cols-2 lg:grid-cols-3">
      <Field as={Search} name="search" label="Căutare" placeholder="Caută {entities}..." />
      {/* Add filter fields based on entity requirements */}
    </div>
  </AutoSubmitForm>
);

export default {Entity}Filters;
```

**{Entity}ActionsCell.jsx** — Edit link + delete with confirmation.

A `cell` component receives the TanStack cell context — `{ getValue, row, column, table }`. There is **no `value` prop**; read the id with `getValue()` (the column's `accessorKey` is `_id`) or `row.original._id`.

```jsx
import { delete{Entity} } from '@api/{entity}';
import { AreYouSure, Button } from '@components';
import { useDisclosure, useMutation } from '@hooks';
import { Pencil, Trash2 } from 'lucide-react';

const {Entity}ActionsCell = ({ getValue, row }) => {
  const id = getValue();
  const { isOpen, show, hide } = useDisclosure();

  const mutation = useMutation(delete{Entity}, { invalidateQueries: 'admin/{entities}' });

  const handleDelete = async () => {
    await mutation.mutateAsync(id);
    hide();
  };

  return (
    <div className="flex items-center justify-center gap-2">
      <Button
        href={`/mgt-portal/admin/{entities}/edit/${id}`}
        className="flex size-8 items-center justify-center rounded-md border border-primary text-primary hover:bg-primary hover:text-white"
        title="Editează"
      >
        <Pencil className="size-3.5" />
      </Button>

      <Button
        onClick={show}
        className="flex size-8 items-center justify-center rounded-md border border-red-600 text-red-600 hover:bg-red-600 hover:text-white"
        title="Șterge"
      >
        <Trash2 className="size-3.5" />
      </Button>

      <AreYouSure
        isOpen={isOpen}
        hide={hide}
        onConfirm={handleDelete}
        title="Confirmare ștergere"
        message={`Sigur vrei să ștergi "${row.original.name}"?`}
        isLoading={mutation.isPending}
      />
    </div>
  );
};

export default {Entity}ActionsCell;
```

**{Entity}StatusCell.jsx** — same context signature; use `Pill` from `@components`.
```jsx
const {Entity}StatusCell = ({ getValue }) => { /* ... */ };
```

**{Entity}FormLoading.jsx / {Entity}FormError.jsx** — skeleton (with `Bone`) and error state for the edit page.

**index.js** — Barrel exports for all components in the directory.

### 9. Form Components — `components/Forms/`

**{Entity}Form.jsx** — Shared form fields using `Field` with field components from `@components/Fields`. Uses `useFormContext` if needed for conditional logic.

**Add{Entity}Form.jsx** — Wraps `{Entity}Form` with `HookForm`/`Form`/`Submit`. Uses `useMutation(create{Entity}, { invalidateQueries: 'admin/{entities}', successCallback: () => router.push('/mgt-portal/admin/{entities}') })`.

**Edit{Entity}Form.jsx** — Same structure, receives `{entity}` as prop, uses `initialValues={{entity}}` and `useMutation(update{Entity})` passing `{ id: {entity}._id, data }`.

### 10. Barrel Export Updates

- `data/index.js` → `export { default as {entity}Columns } from './{entity}-columns';`
- `components/Admin/index.js` → export the new `{Entity}` subfolder
- `components/Forms/index.js` → `Add{Entity}Form` and `Edit{Entity}Form`
- `models/` has no barrel — import directly from `@models/{entity}`

Keep every barrel ordered so a module's dependencies are exported before its consumers.

### 11. Navigation

Add the entry to the admin sidebar (`components/Pages.jsx`, or `AdminNav.jsx` if the project has one) using the `/mgt-portal/admin/...` prefix.

## Important Conventions

- **Max 40-50 lines per file** — keep each file short. If a component grows beyond this, extract sub-components.
- **Separate status components** — `{Entity}FormLoading.jsx` / `{Entity}FormError.jsx` in `components/Admin/{Entity}/`, using `Bone` for skeletons.
- **Lucide icons** — `import { Plus } from 'lucide-react'` then `<Plus className="size-4" />`. Import every icon you use; the snippets above are complete.
- **No arbitrary Tailwind values** — don't use `p-[50px]` syntax.
- **Only colors from `tailwind.config.js`** — check the theme before using a token like `metal-200` or `accent`; they don't exist in every project.
- **Check `examples/` folder first** — `examples/components/Todos/TodoTable.jsx` and `examples/data/todo-columns.js` are the live, working reference for the table system.

## After Generation

1. Verify all imports resolve — especially names taken from another project's admin
2. Check that barrel exports are updated
3. Add the navigation link
4. Confirm the backend endpoints (`/admin/{entities}`) exist and return `{ pages, pageParams }` for lists, `{ data, message }` for mutations
5. Load the list page and confirm rows render — an empty table with a correct row count is the signature of a column-format mismatch
