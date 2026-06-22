---
name: theme-styling
description: >
  Style React Native components with react-native-unistyles v3 — using theme tokens (colors, spacing,
  typography, borderRadius), dynamic style functions, runtime safe-area insets, and light/dark theming.
  Use this skill whenever the user wants to style a component, apply the theme, add dark mode, use
  spacing/color tokens, toggle themes, add a new color or spacing value, or fix styling that uses magic
  numbers or the wrong StyleSheet. Trigger on: "style this", "use the theme", "dark mode", "theme
  toggle", "add a color", "spacing token", "unistyles", or any styling/theming task.
---

# Theme Styling (react-native-unistyles v3)

This starter styles exclusively with **react-native-unistyles v3**. Never import `StyleSheet` from
`react-native`, and never add NativeWind/Tailwind.

```jsx
import { StyleSheet } from 'react-native-unistyles';   // ← the only StyleSheet
```

## Theme tokens

Defined in `constants/theme/` and assembled in `theme.js`:

| Token group | Source | Examples |
|-------------|--------|----------|
| `theme.colors` | `colors.js` | `background`, `surface`, `text`, `textSecondary`, `primary`, `accent`, `card`, `cardBorder`, `btnPrimary`, `btnPrimaryText`, `error`, `success`, `tabBar` |
| `theme.spacing` | `spacing.js` | numeric `1..32` (×4px) **and** named `xxs xs sm md lg xl xl2 xl3 xl4 xl5`, plus `screenEdge`, `gutter` |
| `theme.typography` | `typography.js` | `fontFamily`, `fontSize`, `fontWeight`, `lineHeight`, `letterSpacing`, `textStyles` |
| `theme.borderRadius` | `borderRadius.js` | `xs sm md lg xl round` |

`colors` has `light` and `dark` variants; unistyles swaps the active set by theme. Reference
semantic names (`theme.colors.text`) — never raw hex in components.

## Basic usage

```jsx
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

## Runtime values (`rt`) — safe areas

The second callback arg is the runtime. Use it for insets:

```jsx
const styles = StyleSheet.create((theme, rt) => ({
    wrapper: {
        flex: 1,
        paddingTop: rt.insets.top,
        backgroundColor: theme.colors.background,
    },
}));
```

(`PageContainer` already does this — prefer it for top-level screens.)

## Dynamic styles (function-valued keys)

Return a function from a style key to parametrize it; call it in `style={}`:

```jsx
const styles = StyleSheet.create((theme) => ({
    container: (isActive) => ({
        backgroundColor: isActive ? theme.colors.primary : theme.colors.surface,
        padding: theme.spacing.md,
    }),
}));

// usage
<View style={styles.container(isActive)} />
```

## Light / dark theming

Themes are configured in `constants/theme/unistyles.js` (`adaptiveThemes: false`,
`initialTheme: 'light'`). **`unistyles.js` must load before the app** — `index.js` imports it before
`expo-router/entry`. Don't reorder that.

Toggle at runtime with the `useTheme` hook (wraps `UnistylesRuntime`):

```jsx
import { useTheme } from '@hooks';

const { themeName, toggleTheme } = useTheme();
// toggleTheme() flips light ↔ dark via UnistylesRuntime.setTheme()
```

There's a ready-made `<ThemeToggle />` in `@components`.

## Adding tokens

- **New color**: add it to BOTH `light` and `dark` in `constants/theme/colors.js` with a semantic name
- **New spacing**: extend `constants/theme/spacing.js` (keep the 4px grid; prefer reusing named aliases)
- **New radius/typography**: extend the matching file in `constants/theme/`
- After adding, reference via `theme.<group>.<name>` — never inline the literal

## Text

Don't style text via raw `<Text>` + color. Use `<ThemedText type='…'>`, which pulls from
`theme.typography.textStyles` and `theme.colors.text`. To position text, wrap it in a styled `<View>`
rather than restyling the ThemedText.

## Rules

- Only `react-native-unistyles` `StyleSheet.create((theme, rt) => …)`
- All colors/spacing/radii via tokens — no magic numbers, no hex literals in components
- Style objects: keys ordered logically; the file's `styles` block sits at the bottom, after the
  default export, matching existing components
- Keep components ≤ ~60-70 LOC; trailing empty line
