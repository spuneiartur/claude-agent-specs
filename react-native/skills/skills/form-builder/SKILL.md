---
name: form-builder
description: >
  Build validated forms in React Native using the starter's Forms system — react-hook-form with a yup
  resolver, wrapped by HookForm/Form/Field/Fieldset/Submit. Use this skill whenever the user wants to
  create a form, add validation, build a login/signup/profile/contact form, add input fields with error
  messages, or wire form submission. Trigger on: "create a form", "add a form", "form validation",
  "login form", "signup form", "validated input", "yup schema", or any data-entry surface.
---

# Form Builder

Forms use the `components/Forms` system, which wraps **react-hook-form** with a **yup** resolver.
See `app/examples/form-example.jsx` for a complete working reference.

## The system

| Component | Role |
|-----------|------|
| `HookForm` | Provider. Takes `validationSchema`, `initialValues`, `onSubmit`. Exposes `methods.submitForm`. |
| `Form` | Layout wrapper (`<View>`). Optional `debug` prop renders form state (dev only). |
| `Field` | Connects an input to RHF `Controller`. Pass the input via `as=`, plus `name`/`label`/`help`. |
| `Fieldset` | Label + error/help text around a field (used automatically by `Field` when `label`/`help` set). |
| `Submit` | Submit button (ButtonPrimary). Reads `isSubmitting`, calls `submitForm`. |

Import them from `@components/Forms`.

## Full example

```jsx
import { ThemedText } from '@components';
import { Field, Form, HookForm, Submit } from '@components/Forms';
import { PageContainer } from '@components/ui/PageContainers';
import { ScrollView, TextInput } from 'react-native';
import { StyleSheet } from 'react-native-unistyles';
import * as Yup from 'yup';

const validationSchema = Yup.object().shape({
    email: Yup.string().email('Must be a valid email').required('Email is required'),
    message: Yup.string().required('Message is required'),
});

const initialValues = {
    email: '',
    message: '',
};

const ContactScreen = () => {
    const handleSubmit = (values, methods) => {
        // call an api/ service here; see the api-service skill
        console.log(values);
        methods.reset();
    };

    return (
        <PageContainer>
            <ScrollView style={styles.container}>
                <ThemedText type='title'>Contact</ThemedText>
                <HookForm
                    validationSchema={validationSchema}
                    initialValues={initialValues}
                    onSubmit={handleSubmit}>
                    <Form style={styles.form}>
                        <Field
                            as={TextInput}
                            name='email'
                            label='Email Address'
                            help='Enter your email'
                            placeholder='you@example.com'
                            keyboardType='email-address'
                            autoCapitalize='none'
                            autoCorrect={false}
                        />
                        <Field
                            as={TextInput}
                            name='message'
                            label='Message'
                            placeholder='Your message'
                            multiline
                            numberOfLines={4}
                        />
                        <Submit title='Send' />
                    </Form>
                </HookForm>
            </ScrollView>
        </PageContainer>
    );
};

export default ContactScreen;

const styles = StyleSheet.create((theme) => ({
    container: {
        flex: 1,
        backgroundColor: theme.colors.background,
        padding: theme.spacing.md,
    },
    form: {
        gap: theme.spacing.xs,
    },
}));
```

## How it wires together

- `HookForm` builds `methods = useForm({ resolver: yupResolver(schema), defaultValues: initialValues })`,
  spreads `<FormProvider>`, and attaches `methods.submitForm` (handles validate → submit, and marks all
  fields touched on validation failure so errors show).
- `Field` reads `control` from context and renders the `as` component inside a `Controller`, mapping
  RHF's `onChange`/`onBlur`/`value` onto the input (`onChangeText`/`onChange`/`onBlur`).
- `Fieldset` shows the label, and shows the error message when the field is touched (or the form was
  submitted) and has an error; otherwise it shows `help`.
- `Submit` pulls `isSubmitting` and `submitForm` from context — no `onPress` wiring needed.

## Rules

- Validation schema: `Yup.object().shape({...})` — declare it module-scope above the component
- `initialValues` keys MUST match schema keys and `Field` `name`s
- Submit handler signature is `(values, methods)`; call `methods.reset()` after a successful submit
- Pass the input component via `as=` (e.g. `as={TextInput}`, or a custom `@components/ui/Inputs` input)
- For a field without label/help, `Field` renders the bare controlled input (no fieldset chrome)
- Do real submission through an `api/` service (try/catch + `Toaster`) — see the `api-service` skill
- Keep `debug` off in committed code (or guard with dev check — `Form` already hides it in production)
- File ends with a trailing empty line

## Custom inputs

Instead of RN `TextInput`, you can pass a starter input as `as={...}`:

```jsx
import { TextInput } from '@components/ui/Inputs';

<Field as={TextInput} name='phone' label='Phone' />
```

`components/ui/Inputs` exports `TextInput`, `PhoneInput`, `TextRichInput`. Any input that accepts
`value` + `onChangeText` works with `Field`.
