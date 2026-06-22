---
name: screen-scaffold
description: >
  Create new React Native screens using Expo Router file-based routing — public screens, auth-protected
  screens, tab screens, dynamic routes, and navigator layouts (Stack/Tabs). Use this skill whenever the
  user wants to add a new screen, add a route, create a page, add a tab, add a protected/authenticated
  screen, set up navigation, add a modal, or wire a dynamic detail screen. Trigger on: "new screen",
  "add route", "create page", "add a tab", "protected screen", "detail screen", "[id] route", "add a
  layout", or any new navigable surface in an Expo app.
---

# Screen Scaffold

Create screens using Expo Router's file-based routing. A file in `app/` becomes a route; `_layout.jsx`
defines the navigator for its folder. Route groups in parentheses — `(auth)`, `(protected)`, `(tabs)` —
organize routes WITHOUT adding a URL segment.

**Always check `app/examples/` first** for a working screen close to what you need.

## Routing model

```
app/
  _layout.jsx                  ← root Stack, fonts + auth hydration + splash
  (auth)/
    login.jsx                  → /login
  (protected)/
    _layout.jsx                ← redirects to /login if no token
    (tabs)/
      _layout.jsx              ← Tabs navigator
      index.jsx                → / (first tab)
      explore.jsx              → /explore
  examples/
    _layout.jsx
    [id].jsx                   → /examples/:id
```

| Pattern | File | URL |
|---------|------|-----|
| Screen | `app/about.jsx` | `/about` |
| Folder index | `app/blog/index.jsx` | `/blog` |
| Dynamic param | `app/blog/[slug].jsx` | `/blog/my-post` |
| Group (no URL) | `app/(protected)/profile.jsx` | `/profile` |
| Layout | `app/blog/_layout.jsx` | navigator, not a screen |

## 1. Public screen

```jsx
import { ThemedText } from '@components';
import { PageContainer } from '@components/ui/PageContainers';
import { ScrollView, View } from 'react-native';
import { StyleSheet } from 'react-native-unistyles';

const AboutScreen = () => {
    return (
        <PageContainer>
            <ScrollView style={styles.container}>
                <View style={styles.header}>
                    <ThemedText type='title'>About</ThemedText>
                </View>
            </ScrollView>
        </PageContainer>
    );
};

export default AboutScreen;

const styles = StyleSheet.create((theme) => ({
    container: {
        flex: 1,
        backgroundColor: theme.colors.background,
        padding: theme.spacing.md,
    },
    header: {
        marginBottom: theme.spacing.lg,
    },
}));
```

`PageContainer` already applies `paddingTop: rt.insets.top` (safe area). Use it for top-level screens
instead of `SafeAreaView`.

## 2. Dynamic route screen

`app/items/[id].jsx` → `/items/:id`. Read params with `useLocalSearchParams`.

```jsx
import { ThemedText } from '@components';
import { PageContainer } from '@components/ui/PageContainers';
import { AxiosAuth } from '@lib';
import { useLocalSearchParams } from 'expo-router';
import { useEffect, useState } from 'react';
import { StyleSheet } from 'react-native-unistyles';

const ItemScreen = () => {
    const { id } = useLocalSearchParams();
    const [item, setItem] = useState(null);

    useEffect(() => {
        AxiosAuth.get(`/admin/items/${id}`).then(setItem).catch(() => setItem(null));
    }, [id]);

    return (
        <PageContainer>
            <ThemedText type='title' style={styles.title}>
                {item?.name ?? 'Loading…'}
            </ThemedText>
        </PageContainer>
    );
};

export default ItemScreen;

const styles = StyleSheet.create((theme) => ({
    title: {
        padding: theme.spacing.md,
    },
}));
```

> Data fetching here is illustrative. Prefer extracting requests into an `api/` module
> (see the `api-service` skill) and calling it from the screen.

## 3. Navigator layout (`_layout.jsx`)

A Stack layout for a folder:

```jsx
import { Stack } from 'expo-router';

const ItemsLayout = () => {
    return (
        <Stack screenOptions={{ headerShown: false }}>
            <Stack.Screen name='index' />
            <Stack.Screen name='[id]' options={{ presentation: 'card' }} />
        </Stack>
    );
};

export default ItemsLayout;
```

A Tabs layout uses the custom tab bar and `IconSymbol`:

```jsx
import { IconSymbol } from '@components/Icons';
import { NavigationTabBar } from '@components/ui/TabNavigators';
import { Tabs } from 'expo-router';

export default function TabLayout() {
    return (
        <Tabs
            tabBar={(props) => <NavigationTabBar {...props} />}
            screenOptions={{ headerShown: false }}>
            <Tabs.Screen
                name='index'
                options={{
                    title: 'Home',
                    tabBarIcon: ({ color }) => <IconSymbol size={28} name='house.fill' color={color} />,
                }}
            />
        </Tabs>
    );
}
```

Icon `name`s map SF Symbols → Material Icons in `components/Icons/IconSymbol.jsx`. Add a new
entry to that `MAPPING` before using a new icon name.

## 4. Auth-protected screens

Put screens that require login under `app/(protected)/`. The group's `_layout.jsx` guards them:

```jsx
import { useAuthStore } from '@lib';
import { Redirect, Stack } from 'expo-router';

const ProtectedLayout = () => {
    const isAuthenticated = useAuthStore((state) => !!state.token);

    if (!isAuthenticated) {
        return <Redirect href='/login' />;
    }

    return (
        <Stack>
            <Stack.Screen name='(tabs)' options={{ headerShown: false }} />
        </Stack>
    );
};

export default ProtectedLayout;
```

To add a new protected screen, drop a `.jsx` file inside `(protected)` (or a nested group) and,
if it should appear in the navigator explicitly, register it with `<Stack.Screen name='...' />`.

## 5. Modal / bottom-sheet screen

Use the bottom-sheet nav options from `components/BottomSheets/constants.js`:

```jsx
import { Stack } from 'expo-router';
import { NAVIGATION_BOTTOM_SHEET_NAV_OPTIONS } from '@components/BottomSheets/constants';

<Stack.Screen name='bottom-sheet-content' options={NAVIGATION_BOTTOM_SHEET_NAV_OPTIONS} />
```

## Navigation

```jsx
import { router, useRouter } from 'expo-router';

router.push('/items/42');   // navigate
router.back();              // go back
router.replace('/login');   // replace (no back)
```

Typed routes are enabled (`app.json` → `experiments.typedRoutes`), so route strings are checked.

## Conventions checklist

- Screen file default-exports the component; name it `{Subject}Screen`
- Imports ordered A-Z, `@aliases` over relative paths
- Style with `StyleSheet.create((theme, rt) => …)` from `react-native-unistyles`
- Use `<ThemedText>` not `<Text>`; spacing via `theme.spacing.*`
- Keep files short (≤ ~60-70 LOC); extract data fetching into `api/`
- File ends with a trailing empty line
