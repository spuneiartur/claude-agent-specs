---
name: component-factory
description: >
  Create reusable React Native components following the starter's conventions — unistyles styling,
  barrel exports, the `_core` primitive pattern, ThemedText usage, and feature directories. Use this
  skill whenever the user wants to create a new component, add a UI element, build a card/modal/list
  item/widget, add a design-system primitive (button/input variant), or organize components into a
  feature folder. Trigger on: "create component", "add a component", "new component", "build a card",
  "add a button variant", "make a reusable", or any reusable RN view that needs to be created.
---

# Component Factory

Create React Native components matching the starter's architecture. Styling is **react-native-unistyles
v3** (`StyleSheet.create((theme, rt) => …)`), never `StyleSheet` from `react-native` and never
NativeWind/Tailwind.

**Search `components/` and `components/ui/` for an existing primitive before building a new one.**

## Where components live

| Location | Use for |
|----------|---------|
| `components/` | shared app components (`ThemedText`, `Collapsible`, `LottiePlayer`) |
| `components/ui/` | design-system primitives — Buttons, Inputs, PageContainers, NavigationBars, TabNavigators |
| `components/Forms/` | the react-hook-form system (see `form-builder` skill) |
| `components/{Feature}/` | a group of related components for one feature, with its own `index.js` |

Each folder has a barrel `index.js` of named re-exports.

## 1. Standalone component

`components/{Name}.jsx`:

```jsx
import { ThemedText } from '@components';
import { View } from 'react-native';
import { StyleSheet } from 'react-native-unistyles';

const StatCard = ({ label, value }) => {
    return (
        <View style={styles.card}>
            <ThemedText type='caption'>{label}</ThemedText>
            <ThemedText type='title'>{value}</ThemedText>
        </View>
    );
};

export default StatCard;

const styles = StyleSheet.create((theme) => ({
    card: {
        backgroundColor: theme.colors.card,
        borderColor: theme.colors.cardBorder,
        borderWidth: 1,
        borderRadius: theme.borderRadius.md,
        padding: theme.spacing.md,
        gap: theme.spacing.xs,
    },
}));
```

Then add to the barrel `components/index.js`:

```js
export { default as StatCard } from './StatCard';
```

## 2. Design-system primitive with `_core`

UI primitives separate the raw, unstyled wrapper (`_core/_Name.jsx`) from styled variants
(`NamePrimary.jsx`, `NameOutline.jsx`). This is how Buttons and Inputs are built.

`components/ui/Buttons/_core/_Button.jsx` (raw, behavior only):

```jsx
import { TouchableOpacity } from 'react-native';

const _Button = ({ children, ...props }) => {
    return <TouchableOpacity {...props}>{children}</TouchableOpacity>;
};

export default _Button;
```

`components/ui/Buttons/ButtonPrimary.jsx` (styled variant):

```jsx
import { Text } from 'react-native';
import { StyleSheet } from 'react-native-unistyles';
import _Button from './_core/_Button';

const ButtonPrimary = ({ Left, children, Right, disabled, ...props }) => {
    return (
        <_Button style={styles.container(!!disabled)} disabled={disabled} {...props}>
            {Left}
            <Text style={styles.text}>{children}</Text>
            {Right}
        </_Button>
    );
};

export default ButtonPrimary;

const styles = StyleSheet.create((theme) => ({
    container: (isDisabled) => ({
        flexDirection: 'row',
        alignItems: 'center',
        justifyContent: 'center',
        backgroundColor: isDisabled ? theme.colors.neutral : theme.colors.btnPrimary,
        height: 52,
        borderRadius: theme.borderRadius.xl,
        paddingHorizontal: theme.spacing.xl,
        gap: theme.spacing.sm,
    }),
    text: {
        color: theme.colors.btnPrimaryText,
        fontFamily: theme.typography.fontFamily.medium,
        fontSize: theme.typography.fontSize.md,
    },
}));
```

`Left`/`Right` slot props are the convention for icons/spinners around content. The dynamic
`container(isDisabled)` style shows the function-style key pattern.

Barrel `components/ui/Buttons/index.js`:

```js
export { default as ButtonPrimary } from './ButtonPrimary';
```

## 3. Feature directory

For a feature with several parts:

```
components/
  ItemList/
    ItemCard.jsx
    ItemListHeader.jsx
    index.js
```

`components/ItemList/index.js`:

```js
export { default as ItemCard } from './ItemCard';
export { default as ItemListHeader } from './ItemListHeader';
```

## Hard rules

- **Style only with `react-native-unistyles`** `StyleSheet.create((theme, rt) => …)` — use `theme`
  tokens (`colors`, `spacing`, `typography`, `borderRadius`), never magic numbers
- `rt.insets.top` for safe-area padding when needed
- Use **`<ThemedText type='…'>`** for text, not `<Text>`. Do NOT pass `style` that restyles
  ThemedText's color/size — if you must position it, wrap in a `<View>` and style the View.
  (Raw `<Text>` is acceptable *inside* a primitive like a button label, as the existing code does.)
- One default export per component file; barrel `index.js` uses named re-exports
- Keep files ≤ ~60-70 LOC, ≤ 2 levels of nesting
- Imports ordered A-Z, `@aliases` over relative paths (`./_core/_Button` relative within a primitive folder is fine)
- Slot props for composition use PascalCase: `Left`, `Right`
- File ends with a trailing empty line
- Do NOT add new npm packages unless told to

## ThemedText types

`default`, `defaultSemiBold`, `title`, `subtitle`, `link`, `heading1`, `heading2`, `heading3`,
`caption`, `overline`. These map to `theme.typography.textStyles`.
