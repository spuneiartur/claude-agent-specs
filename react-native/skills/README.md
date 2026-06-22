# React Native + Expo Skills

Claude agent skills that teach repeatable development patterns for the **resto-native** starter (React Native 0.79+, Expo SDK 53+, expo-router, react-native-unistyles v3, zustand, react-hook-form + yup, axios).

Each skill is self-contained in `skills/<name>/SKILL.md` with optional `references/`.

## Skills

| Skill | Description |
|-------|-------------|
| screen-scaffold | Create expo-router screens (public, protected, tabs, dynamic routes, layouts) |
| component-factory | Generate RN components with unistyles, barrel exports, and the `_core` primitive pattern |
| form-builder | Build validated forms with the `Forms` system (HookForm/Form/Field/Submit) + yup |
| api-service | Create `api/` request modules using `Axios`/`AxiosAuth`, the auth store, and `Toaster` |
| data-fetching | Fetch/mutate screen data with the TanStack Query hooks (`useQuery`/`useMutation`/`useInfiniteQuery`) |
| theme-styling | Style with `react-native-unistyles`, theme tokens, and light/dark theming |
| skill-creator | Create new skills from scratch |

## Usage

Copy the `CLAUDE.md` and `skills/` directory into a React Native + Expo project, or register this folder as a Claude Code plugin marketplace:

```
/plugin marketplace add ./react-native/skills
```

These skills assume the conventions documented in the sibling `CLAUDE.md`.
