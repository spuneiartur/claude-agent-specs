# Admin CRUD Patterns Reference

Exact code templates for generating a new admin entity. These match the live implementations in `examples/components/Todos/` and `examples/data/todo-columns.js` — when in doubt, read those files, they are the running reference.

## API Service Pattern

**Source:** `api/article.js` · exports ordered A-Z

```js
import { axiosAuth } from '@lib';

export const createArticle = (data) => {
  return axiosAuth.post('/admin/articles', data);
};

export const deleteArticle = (id) => {
  return axiosAuth.delete(`/admin/articles/${id}`);
};

export const getArticle = (id) => {
  return axiosAuth.get(`/admin/articles/${id}`);
};

export const updateArticle = ({ id, data }) => {
  return axiosAuth.put(`/admin/articles/${id}`, data);
};
```

## Validation Model Pattern

**Source:** `models/article.js`

```js
import * as Yup from 'yup';

export const validationSchema = Yup.object().shape({
  title: Yup.string().required('Titlul este obligatoriu'),
  slug: Yup.string()
    .required('Slug-ul este obligatoriu')
    .matches(/^[a-z0-9_-]+$/, 'Slug-ul poate conține doar litere mici, cifre, cratime și underscore')
    .min(2, 'Slug-ul trebuie să aibă cel puțin 2 caractere')
    .max(100, 'Slug-ul nu poate depăși 100 de caractere'),
  status: Yup.string().oneOf(['draft', 'published']).required('Statusul este obligatoriu'),
  // Add entity-specific fields here
});

export const initialValues = {
  title: '',
  slug: '',
  status: 'published',
  // Match all fields from validationSchema
};
```

Keep the messages identical to the backend Yup schema for the same field — otherwise the user sees two different wordings for one rule.

## Filter Model Pattern

**Source:** `models/article-filters.js`

```js
import * as Yup from 'yup';

export const validationSchema = Yup.object().shape({
  search: Yup.string(),
  status: Yup.string(),
  created_from: Yup.string(),
  created_to: Yup.string(),
});

export const initialValues = {
  search: '',
  status: '',
  created_from: '',
  created_to: '',
};
```

## Table Columns Pattern — TanStack Table v8

**Source:** `examples/data/todo-columns.js`

```js
import { Time } from '@components';
import { EntityActionsCell, EntityStatusCell } from '@components/Admin/Entity';

const entityColumns = [
  {
    id: 'name',
    header: 'Nume',
    accessorKey: 'name',
    extraClass: 'font-medium text-gray-900',
  },
  {
    id: 'author',
    header: 'Autor',
    accessorKey: 'identity.name',   // dot notation for nested values
  },
  {
    id: 'status',
    header: 'Status',
    accessorKey: 'status',
    cell: EntityStatusCell,
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
    cell: EntityActionsCell,
    extraClass: 'w-32 text-center',
  },
];

export default entityColumns;
```

### Why the shape matters

`TableSuccess.jsx` hands the array straight to `useReactTable`, then:

```jsx
// TableHeader.jsx
flexRender(column.columnDef.header, column.getContext?.())
// TableRow.jsx
flexRender(cell.column.columnDef.cell, cell.getContext())
```

v6 keys (`Header`, `accessor`, `Cell`) are not part of `columnDef` in v8. They are ignored without warning — the table renders the right number of rows with blank headers and blank cells.

`extraClass` is a project-specific extra read by `TableHeader.jsx` for column widths. TanStack passes unknown keys through untouched.

## Cell Component Pattern

Every `cell` component receives the TanStack **cell context**, not a `value` prop:

```
{ getValue, row, column, table, cell, renderValue, getContext }
```

- `getValue()` — the value at this column's `accessorKey`
- `row.original` — the full document, for reading sibling fields

```jsx
import { Pill } from '@components';

const statusConfig = {
  published: { label: 'Publicat', className: 'bg-green-50 text-green-700 border-green-200' },
  draft: { label: 'Ciornă', className: 'bg-yellow-50 text-yellow-700 border-yellow-200' },
};

const EntityStatusCell = ({ getValue }) => {
  const config = statusConfig[getValue()] ?? {
    label: 'Necunoscut',
    className: 'bg-gray-50 text-gray-700 border-gray-200',
  };

  return <Pill className={`border ${config.className}`}>{config.label}</Pill>;
};

export default EntityStatusCell;
```

Components reused as cells must follow the same contract. `Time` from `@components` already does — that is why it can be dropped straight into a column.

## Add Form Pattern

**Source:** `components/Forms/AddArticleForm.jsx`

```jsx
import { createEntity } from '@api/entity';
import { Button } from '@components';
import { EntityForm } from '@components/Forms';
import { Form, HookForm, Submit } from '@components/HookForm';
import { useMutation } from '@hooks';
import { initialValues, validationSchema } from '@models/entity';
import { useRouter } from 'next/router';

const AddEntityForm = () => {
  const router = useRouter();
  const mutation = useMutation(createEntity, {
    invalidateQueries: 'admin/entities',
    successCallback: () => router.push('/mgt-portal/admin/entities'),
  });

  const handleSubmit = async (data) => mutation.mutateAsync(data);

  return (
    <HookForm
      initialValues={initialValues}
      validationSchema={validationSchema}
      onSubmit={handleSubmit}
    >
      <Form>
        <div className="mb-6 rounded-lg border border-gray-200 bg-white shadow-sm">
          <div className="flex items-center justify-between px-6 py-4">
            <p className="text-sm text-gray-600">
              {mutation.isPending ? 'Se salvează...' : 'Gata de salvare?'}
            </p>
            <div className="flex items-center gap-3">
              <Button
                onClick={() => router.push('/mgt-portal/admin/entities')}
                className="rounded-md px-4 py-2 font-medium text-gray-600 hover:bg-gray-100"
              >
                Anulează
              </Button>
              <Submit disabled={mutation.isPending}>
                {mutation.isPending ? 'Se salvează...' : 'Creează'}
              </Submit>
            </div>
          </div>
        </div>

        <EntityForm />
      </Form>
    </HookForm>
  );
};

export default AddEntityForm;
```

`Button` renders `<button type="button">` by default, so the Cancel button will not submit the form.

## Edit Form Pattern

**Source:** `components/Forms/EditArticleForm.jsx`

```jsx
import { updateEntity } from '@api/entity';
import { Button } from '@components';
import { Form, HookForm, Submit } from '@components/HookForm';
import { useMutation } from '@hooks';
import { validationSchema } from '@models/entity';
import { useRouter } from 'next/router';
import EntityForm from './EntityForm';

const EditEntityForm = ({ entity }) => {
  const router = useRouter();
  const mutation = useMutation(updateEntity, {
    invalidateQueries: 'admin/entities',
    successCallback: () => router.push('/mgt-portal/admin/entities'),
  });

  const handleSubmit = async (data) => mutation.mutateAsync({ id: entity._id, data });

  return (
    <HookForm initialValues={entity} validationSchema={validationSchema} onSubmit={handleSubmit}>
      <Form>
        <div className="mb-6 rounded-lg border border-gray-200 bg-white shadow-sm">
          <div className="flex items-center justify-between px-6 py-4">
            <p className="text-sm text-gray-600">
              {mutation.isPending ? 'Se actualizează...' : 'Gata de salvare?'}
            </p>
            <div className="flex items-center gap-3">
              <Button
                onClick={() => router.push('/mgt-portal/admin/entities')}
                className="rounded-md px-4 py-2 font-medium text-gray-600 hover:bg-gray-100"
              >
                Anulează
              </Button>
              <Submit disabled={mutation.isPending}>
                {mutation.isPending ? 'Se actualizează...' : 'Actualizează'}
              </Submit>
            </div>
          </div>
        </div>

        <EntityForm />
      </Form>
    </HookForm>
  );
};

export default EditEntityForm;
```

## Shared Form Fields Pattern

**Source:** `components/Forms/ArticleForm.jsx`

```jsx
import { Dropdown, Input, Textarea } from '@components/Fields';
import { Field } from '@components/HookForm';

const statusOptions = [
  { value: 'draft', label: 'Ciornă' },
  { value: 'published', label: 'Publicat' },
];

const EntityForm = () => {
  return (
    <div className="rounded-lg border border-gray-200 bg-white shadow-sm">
      <div className="space-y-4 p-6">
        <Field as={Input} name="title" label="Titlu" placeholder="Introdu titlul" />
        <Field as={Input} name="slug" label="Slug" placeholder="url-slug" />
        <Field as={Textarea} name="description" label="Descriere" rows={4} />
        <Field as={Dropdown} name="status" label="Status">
          {statusOptions.map((option) => (
            <option key={option.value} value={option.value}>
              {option.label}
            </option>
          ))}
        </Field>
      </div>
    </div>
  );
};

export default EntityForm;
```

`Field` sets `id` from `name` automatically — you don't need to pass it. Only call `useFormContext()` when you actually need to watch another field.

There is no `SlugInput` component in the starter. If the project needs auto-slug-from-title, build it as a project component (`components/Fields/SlugInput.jsx`) using `useFormContext` + `watch('title')`; don't import it as if it already existed.

## Key Import Paths

| What | Import From |
|------|------------|
| axiosAuth | `@lib` |
| checkAuth, withAuth | `@auth` |
| AreYouSure, Bone, Button, Layout, Pill, Time | `@components` |
| ArrayField, AutoSubmitForm, Field, Form, HookForm, Submit | `@components/HookForm` |
| DatePicker, Dropdown, Input, Search, Textarea | `@components/Fields` |
| TableColumns, TableError, TableLoading, TableSuccess | `@components/Tables` |
| LoadMoreOnClick, LoadMoreOnScroll | `@components/Buttons` |
| useDisclosure, useInfiniteQuery, useMutation, useQuery | `@hooks` |
| useFormContext | `react-hook-form` |
| useRouter | `next/router` |
| Icons | `lucide-react` |
| Yup | `yup` |

## Gotchas

1. **React Query v5 status is `'pending'`, not `'loading'`.** A `status === 'loading'` branch never runs, so the skeleton never shows and the page looks broken while fetching.
2. **`useInfiniteQuery` already flattens.** Its `select` returns `{ pages, pageParams }` where `pages` is a flat array of documents. Pass `data` to `TableSuccess` as-is.
3. **Cell components get the TanStack context**, not `{ value, row: { original } }`. Use `getValue()` and `row.original`.
4. **`enabled: Boolean(id)`** on the edit page — `router.query` is empty on first render of a dynamic route.
5. **Admin hrefs need the `/mgt-portal` prefix** when `next.config.js` rewrites `/mgt-portal/admin/:path*` → `/admin/:path*` and redirects `/admin/:path*` → `/404`. Check before generating.
6. **`important: true` in `tailwind.config.js`** — every utility emits `!important`. Custom CSS overrides need higher specificity.
7. **Colors are per project.** `metal-*` and `accent` exist in some projects and not in others. Read `tailwind.config.js`.
