---
name: form-builder
description: >
  Build forms using React Hook Form with Yup validation, following the project's HookForm component
  system and Field components. Use this skill whenever the user needs to create a form (contact,
  settings, login, data entry, filters, any form), add form validation, use the project's
  Field/HookForm components, create filter forms, or build any data input interface. Also trigger
  when the user mentions "form", "validation", "Yup schema", "fields", "input fields", "form
  submission", or wants to collect user data through a UI.
---

# Form Builder

Build forms using the project's React Hook Form + Yup validation system. This project has a complete form infrastructure — use it instead of building forms from scratch.

## Architecture Overview

The form system has three layers:

1. **Validation model** (`models/{name}.js`) — Yup schema + initial values
2. **HookForm wrapper** (`@components/HookForm`) — Connects React Hook Form with Yup resolver
3. **Field components** (`@components/Fields`) — Pre-built input components with error display

## Step 1: Create the Validation Model

Create `models/{form-name}.js` with two named exports:

```js
import * as Yup from 'yup';

export const validationSchema = Yup.object().shape({
  name: Yup.string().required('Name is required'),
  email: Yup.string().email('Invalid email').required('Email is required'),
  status: Yup.string().oneOf(['active', 'inactive']).required(),
  description: Yup.string(),
  image: Yup.object().nullable(),
  items: Yup.array().of(
    Yup.object().shape({
      title: Yup.string().required(),
      value: Yup.mixed().required(),
    })
  ),
});

export const initialValues = {
  name: '',
  email: '',
  status: 'active',
  description: '',
  image: null,
  items: [],
};
```

Every field in `validationSchema` must have a matching key in `initialValues`.

For **filter forms**, create `models/{entity}-filters.js` — all fields are optional strings:

```js
import * as Yup from 'yup';

export const validationSchema = Yup.object().shape({
  search: Yup.string(),
  status: Yup.string(),
  created_from: Yup.string(),
  created_to: Yup.string(),
});

export const initialValues = { search: '', status: '', created_from: '', created_to: '' };
```

## Step 2: Build the Form Component

### Standard Form (manual submit)

```jsx
import { create{Entity} } from '@api/{entity}';
import { Button } from '@components';
import { Form, HookForm, Submit } from '@components/HookForm';
import { useMutation } from '@hooks';
import { initialValues, validationSchema } from '@models/{entity}';
import { useRouter } from 'next/router';

const Add{Entity}Form = () => {
  const router = useRouter();
  const mutation = useMutation(create{Entity}, {
    invalidateQueries: 'admin/{entities}',
    successCallback: () => router.push('/admin/{entities}'),
  });

  const handleSubmit = async (data) => mutation.mutateAsync(data);

  return (
    <HookForm initialValues={initialValues} validationSchema={validationSchema} onSubmit={handleSubmit}>
      <Form>
        {/* Form fields go here (see Step 3) */}
        <Submit disabled={mutation.isPending}>
          {mutation.isPending ? 'Saving...' : 'Save'}
        </Submit>
      </Form>
    </HookForm>
  );
};
```

### Filter Form (auto-submit on change)

```jsx
import { Search, Dropdown, DatePicker } from '@components/Fields';
import { Field } from '@components/HookForm';
import AutoSubmitFilterForm from '@components/HookForm/AutoSubmitFilterForm';
import { initialValues, validationSchema } from '@models/{entity}-filters';

const Filters = ({ setOptions }) => (
  <AutoSubmitFilterForm initialValues={initialValues} validationSchema={validationSchema} onSubmit={setOptions}>
    <div className="grid grid-cols-1 gap-4 md:grid-cols-2 lg:grid-cols-3">
      <Field as={Search} name="search" placeholder="Search..." label="Search" />
      <Field as={Dropdown} name="status" label="Status">
        <option value="">All</option>
        <option value="active">Active</option>
      </Field>
      <Field as={DatePicker} name="created_from" label="From date" />
    </div>
  </AutoSubmitFilterForm>
);
```

### Edit Form (pre-populated with existing data)

Same as the standard form, but pass the entity object as `initialValues`:

```jsx
const EditForm = ({ entity }) => (
  <HookForm initialValues={entity} validationSchema={validationSchema} onSubmit={handleSubmit}>
    ...
  </HookForm>
);
```

## Step 3: Use Field Components

The `<Field>` component renders any field type via the `as` prop:

```jsx
<Field as={Input} name="title" label="Title" placeholder="Enter title" required />
```

### Available Field Components

Import from `@components/Fields`:

| Component | Use For |
|-----------|---------|
| `Input` | Text, generic input |
| `Email` | Email with validation styling |
| `Password` | Password with toggle visibility |
| `Phone` | Phone number |
| `Number` | Numeric input |
| `Textarea` | Multi-line text |
| `RichText` | Rich text editor (React Quill) |
| `Dropdown` | Select with `<option>` children |
| `Search` | Search input with debounce |
| `DatePicker` | Date selection |
| `TimePicker` | Time selection |
| `DateOfBirth` | Date of birth (day/month/year dropdowns) |
| `Checkbox` | Single checkbox |
| `Toggle` | Toggle switch |
| `Radio` | Radio button (use within `RadioGroup`) |
| `RadioGroup` | Group of radio buttons |
| `Select` | Custom select |
| `LabeledSelect` | Select with label |
| `Combobox` | Searchable dropdown |
| `AsyncCombobox` | Searchable dropdown with async data |
| `ComboboxWithImage` | Combobox with image previews |
| `AsyncComboboxWithImage` | Async combobox with images |
| `Autocomplete` | Autocomplete text input |
| `AutocompleteWithImage` | Autocomplete with images |
| `MultiSelectAsync` | Multi-select with async options |
| `AsyncDropdown` | Dropdown loaded from API |
| `SlugInput` | URL slug auto-generated from another field |
| `FileDrop` | Drag-and-drop file upload |
| `FileUpload` | File upload button |
| `Embed` | Embed URL input |
| `PlusMinus` | Increment/decrement number |
| `Recaptcha` | Google reCAPTCHA |

### Special Components

**SlugInput** — Auto-generates URL slug from a source field:
```jsx
<SlugInput sourceField="title" placeholder="url-slug" required />
```

**ArrayField** — Dynamic repeatable sections:
```jsx
import { ArrayField } from '@components/HookForm';

<ArrayField
  name="sections"
  AddComponent={AddButton}
  SectionComponent={SectionRenderer}
  emptyRow={{ type: 'text', content: '', timestamp: Date.now() }}
/>
```

**Fieldset** — Group fields with a label:
```jsx
import { Fieldset } from '@components/HookForm';
<Fieldset legend="Contact Info">...</Fieldset>
```

## Step 4: Access Form State in Nested Components

Use `useFormContext()` from `react-hook-form` to access form state in child components:

```jsx
import { useFormContext } from 'react-hook-form';

const NestedComponent = () => {
  const { formState, watch, setValue } = useFormContext();
  const status = watch('status');
  // ...
};
```

## Key Import Map

| What | Import From |
|------|------------|
| HookForm, Form, Submit, Field, ArrayField, Fieldset | `@components/HookForm` |
| AutoSubmitFilterForm | `@components/HookForm/AutoSubmitFilterForm` |
| All field components | `@components/Fields` |
| useMutation | `@hooks` |
| useFormContext | `react-hook-form` |
| Yup | `yup` |
