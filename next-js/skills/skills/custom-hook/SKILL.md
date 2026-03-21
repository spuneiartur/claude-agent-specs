---
name: custom-hook
description: >
  Create custom React hooks following the project's conventions for naming, file structure, and
  barrel exports. Use this skill whenever the user needs to add a reusable hook, create a custom
  data-fetching hook, encapsulate stateful logic, add a utility hook, or extract repeated logic
  into a hook. Trigger on: "create hook", "custom hook", "useXxx", "add a hook", "extract into
  a hook", or when the user describes reusable stateful logic that should be shared across components.
---

# Custom Hook

Create custom React hooks following the project's established patterns. The project has 21+ hooks in `/hooks/` with a consistent structure.

## File Convention

- **File name:** `use-{name}.js` (kebab-case, `.js` extension — not `.jsx`)
- **Location:** `/hooks/use-{name}.js`
- **Export:** `export default` of the hook function (no named exports)

## Step 1: Create the Hook

```js
// hooks/use-{name}.js

import { useState, useEffect, useCallback } from 'react';

const use{Name} = (initialValue) => {
  const [state, setState] = useState(initialValue);

  // Hook logic here

  return { state, setState };  // Return an object for named destructuring
};

export default use{Name};
```

## Step 2: Register in Barrel Export

Add to `hooks/index.js`:

```js
export { default as use{Name} } from './use-{name}';
```

This enables importing via `import { use{Name} } from '@hooks';`.

## Hook Patterns in This Project

### Pattern 1: Library Wrapper Hook

Wraps a third-party hook with project-specific defaults and behavior. Examples: `useQuery`, `useMutation`, `useInfiniteQuery`.

```js
// Wraps react-query's useMutation with auto toast + redirect
import { toaster } from '@lib';
import { useRouter } from 'next/router';
import { useQueryClient, useMutation as useQueryMutation } from 'react-query';

const useMutation = (fn, options = {}) => {
  const { successCallback, errorCallback, redirectOnSuccess, invalidateQueries, ...rest } = options;
  const router = useRouter();
  const queryClient = useQueryClient();

  return useQueryMutation(fn, {
    onSuccess: (data) => {
      if (invalidateQueries) queryClient.invalidateQueries(invalidateQueries);
      if (data?.message) toaster.success(data.message);
      if (redirectOnSuccess) router.push(redirectOnSuccess);
      if (typeof successCallback === 'function') successCallback();
    },
    onError: (err) => {
      if (err?.message) toaster.error(err.message);
      if (typeof errorCallback === 'function') errorCallback();
    },
    ...rest,
  });
};

export default useMutation;
```

### Pattern 2: Simple State Hook

Manages a small, focused piece of UI state. Examples: `useDisclosure`, `useToggle`.

```js
// useDisclosure - manages open/close state for modals, dropdowns, etc.
import { useState, useCallback } from 'react';

const useDisclosure = (initial = false) => {
  const [isOpen, setIsOpen] = useState(initial);
  const show = useCallback(() => setIsOpen(true), []);
  const hide = useCallback(() => setIsOpen(false), []);
  const toggle = useCallback(() => setIsOpen((prev) => !prev), []);

  return { isOpen, show, hide, toggle };
};

export default useDisclosure;
```

### Pattern 3: Effect-Based Hook

Uses `useEffect` with cleanup. Examples: `useDebounce`, `useOnClickOutside`, `useIntersectionObserver`.

```js
// useDebounce - debounces a value by delay
import { useState, useEffect } from 'react';

const useDebounce = (value, delay = 300) => {
  const [debouncedValue, setDebouncedValue] = useState(value);

  useEffect(() => {
    const timer = setTimeout(() => setDebouncedValue(value), delay);
    return () => clearTimeout(timer);
  }, [value, delay]);

  return debouncedValue;
};

export default useDebounce;
```

### Pattern 4: Data Manipulation Hook

Provides utilities for working with data structures. Examples: `useArray`.

```js
// useArray - array manipulation utilities
import { useState, useCallback } from 'react';

const useArray = (initial = []) => {
  const [value, setValue] = useState(initial);
  const push = useCallback((item) => setValue((arr) => [...arr, item]), []);
  const remove = useCallback((index) => setValue((arr) => arr.filter((_, i) => i !== index)), []);
  const clear = useCallback(() => setValue([]), []);

  return { value, setValue, push, remove, clear };
};

export default useArray;
```

## Guidelines

1. **Return objects, not arrays** — enables named destructuring: `const { isOpen, show } = useDisclosure()`
2. **Accept configuration via parameters** — sensible defaults, override when needed
3. **Memoize callbacks** with `useCallback` to prevent unnecessary re-renders
4. **Clean up effects** — return cleanup functions from `useEffect`
5. **Keep hooks focused** — one hook, one responsibility
6. **Check existing hooks first** — the project already provides: `useArray`, `useChildren`, `useCollapsible`, `useCombobox`, `useDebounce`, `useDisclosure`, `useDropdown`, `useInfiniteQuery`, `useIntersectionObserver`, `useMutation`, `useObserver`, `useOnClickOutside`, `useProfile`, `useQuery`, `useRerender`, `useScrollRestoration`, `useScrollReveal`, `useSelect`, `useSlugGenerator`, `useSwipeable`, `useToggle`
