# Admin CRUD Patterns Reference

This file contains the exact code patterns extracted from the article CRUD implementation. Use these as templates when generating files for a new entity.

## API Service Pattern

**Source:** `api/article.js`

```js
import { axiosAuth } from '@lib';

export const getArticle = (id) => {
  return axiosAuth.get(`/admin/articles/${id}`);
};

export const createArticle = (data) => {
  return axiosAuth.post('/admin/articles', data);
};

export const updateArticle = ({ id, data }) => {
  return axiosAuth.put(`/admin/articles/${id}`, data);
};

export const deleteArticle = (id) => {
  return axiosAuth.delete(`/admin/articles/${id}`);
};
```

## Validation Model Pattern

**Source:** `models/article.js`

```js
import * as Yup from 'yup';

export const validationSchema = Yup.object().shape({
  title: Yup.string().required('Title is required'),
  slug: Yup.string()
    .required('Slug is required')
    .matches(/^[a-z0-9_-]+$/, 'Slug can only contain lowercase letters, numbers, hyphens and underscores')
    .min(2, 'Slug must be at least 2 characters')
    .max(100, 'Slug cannot exceed 100 characters'),
  status: Yup.string().oneOf(['draft', 'published']).required('Status is required'),
  // Add entity-specific fields here
});

export const initialValues = {
  title: '',
  slug: '',
  status: 'published',
  // Match all fields from validationSchema
};
```

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

## Table Columns Pattern

**Source:** `data/article-columns.js`

```js
import { FormattedTime } from '@components';
import { EntityActionsCell, EntityStatusCell } from '@components/Admin/Entity';

const entityColumns = [
  {
    Header: 'Name',
    accessor: 'name',
    extraClass: 'font-medium text-gray-900',
  },
  {
    Header: 'Status',
    accessor: 'status',
    Cell: EntityStatusCell,
    extraClass: 'w-24 text-center font-medium text-gray-900',
  },
  {
    Header: 'Created',
    accessor: 'createdAt',
    Cell: FormattedTime,
    extraClass: 'w-32 font-medium text-gray-900',
  },
  {
    Header: 'Actions',
    accessor: '_id',
    Cell: EntityActionsCell,
    extraClass: 'w-32 text-center font-medium text-gray-900',
  },
];

export default entityColumns;
```

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
    successCallback: () => router.push('/admin/entities'),
  });

  const handleSubmit = async (data) => mutation.mutateAsync(data);

  return (
    <HookForm initialValues={initialValues} validationSchema={validationSchema} onSubmit={handleSubmit}>
      <Form>
        <div className="bg-white border border-gray-200 rounded-lg shadow-sm mb-6">
          <div className="px-6 py-4">
            <div className="flex items-center justify-between">
              <div className="text-sm text-gray-600">
                {mutation.isPending ? 'Saving...' : 'Ready to save?'}
              </div>
              <div className="flex items-center space-x-3">
                <Button onClick={() => router.push('/admin/entities')} className="px-4 py-2 text-gray-600 hover:text-gray-800 hover:bg-gray-100 rounded-md transition-colors font-medium">
                  Cancel
                </Button>
                <Submit disabled={mutation.isPending}>
                  {mutation.isPending ? 'Saving...' : 'Create Entity'}
                </Submit>
              </div>
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
    successCallback: () => router.push('/admin/entities'),
  });

  const handleSubmit = async (data) => mutation.mutateAsync({ id: entity._id, data });

  return (
    <HookForm initialValues={entity} validationSchema={validationSchema} onSubmit={handleSubmit}>
      <Form>
        <div className="bg-white border border-gray-200 rounded-lg shadow-sm mb-6">
          <div className="px-6 py-4">
            <div className="flex items-center justify-between">
              <div className="text-sm text-gray-600">
                {mutation.isPending ? 'Updating...' : 'Ready to save changes?'}
              </div>
              <div className="flex items-center space-x-3">
                <Button onClick={() => router.push('/admin/entities')} className="px-4 py-2 text-gray-600 hover:text-gray-800 hover:bg-gray-100 rounded-md transition-colors font-medium">
                  Cancel
                </Button>
                <Submit disabled={mutation.isPending}>
                  {mutation.isPending ? 'Updating...' : 'Update Entity'}
                </Submit>
              </div>
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
import { Input, Textarea, Dropdown, SlugInput } from '@components/Fields';
import { Field } from '@components/HookForm';
import { useFormContext } from 'react-hook-form';

const statusOptions = [
  { value: 'draft', label: 'Draft' },
  { value: 'published', label: 'Published' },
];

const EntityForm = () => {
  const { formState } = useFormContext();

  return (
    <div className="bg-white rounded-lg shadow-sm border border-gray-200">
      <div className="p-6 space-y-4">
        <Field as={Input} name="title" label="Title" placeholder="Enter title" required />
        <SlugInput sourceField="title" placeholder="url-slug" required />
        <Field as={Dropdown} name="status" label="Status">
          {statusOptions.map((option) => (
            <option key={option.value} value={option.value}>{option.label}</option>
          ))}
        </Field>
        {/* Add entity-specific fields */}
      </div>
    </div>
  );
};

export default EntityForm;
```

## Key Import Paths

| What | Import From |
|------|------------|
| axiosAuth | `@lib` |
| checkAuth, withAuth | `@auth` |
| Layout, Button, AreYouSure | `@components` |
| Field, Form, HookForm, Submit, ArrayField | `@components/HookForm` |
| AutoSubmitFilterForm | `@components/HookForm/AutoSubmitFilterForm` |
| Input, Textarea, Dropdown, Search, DatePicker, SlugInput | `@components/Fields` |
| TableColumns, TableLoading, TableError, TableSuccess | `@components/Tables` |
| LoadMoreOnClick | `@components/Buttons` |
| useQuery, useInfiniteQuery, useMutation, useDisclosure | `@hooks` |
| FormattedTime | `@components` |
| useFormContext | `react-hook-form` |
| useRouter | `next/router` |
| Yup | `yup` |
