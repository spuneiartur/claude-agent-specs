---
name: component-factory
description: >
  Create reusable React components following the project's conventions for naming, file structure,
  barrel exports, Tailwind styling, and hook patterns. Use this skill whenever the user wants to
  create a new component, add a UI element, build a feature section, create a card/modal/widget,
  add a table cell renderer, or organize components into feature directories. Trigger on: "create
  component", "add a component", "new component", "build a widget", "create a card", "add a
  section", or when the user describes any reusable UI element that needs to be created.
---

# Component Factory

Create React components following the project's established conventions. The project has a consistent component architecture — follow it to maintain codebase coherence.

## Component Types

### 1. Standalone Component

For general-purpose, reusable components. Lives directly in `/components/`.

**File:** `components/{ComponentName}.jsx`

```jsx
const ComponentName = ({ prop1, prop2, className = '' }) => {
  return (
    <div className={`base-styles ${className}`}>
      {/* Component content */}
    </div>
  );
};

export default ComponentName;
```

**Then update the barrel export** in `components/index.js`:
```js
export { default as ComponentName } from './ComponentName';
```

### 2. Feature Component Directory

For a group of related components (e.g., a feature section with multiple parts). Lives in `/components/{FeatureName}/`.

```
components/
  FeatureName/
    FeatureCard.jsx
    FeatureList.jsx
    FeatureHeader.jsx
    index.js          ← barrel exports
```

**Barrel export** (`components/{FeatureName}/index.js`):
```js
export { default as FeatureCard } from './FeatureCard';
export { default as FeatureList } from './FeatureList';
export { default as FeatureHeader } from './FeatureHeader';
```

**Import pattern:**
```js
import { FeatureCard, FeatureList } from '@components/FeatureName';
```

### 3. Table Cell Component

For rendering custom cells in admin tables. Lives in `/components/TableCells/`.

**Props contract:** Receives `{ value, row: { original } }` from react-table:

```jsx
const ResourceNameCell = ({ value, row: { original } }) => {
  return (
    <div className="flex items-center space-x-3">
      <span className="font-medium text-gray-900">{value}</span>
      {original.subtitle && (
        <span className="text-sm text-gray-500">{original.subtitle}</span>
      )}
    </div>
  );
};

export default ResourceNameCell;
```

### 4. Admin Feature Components

For admin section components. Lives in `/components/Admin/{Entity}/`.

Standard set per entity:
- `{Entity}Header.jsx` — page header with title, add button, filters
- `{Entity}Table.jsx` — data table with infinite query
- `{Entity}Filters.jsx` — filter form
- `{Entity}ActionsCell.jsx` — edit/delete buttons for table rows
- `{Entity}StatusCell.jsx` — status badge renderer (optional)
- `index.js` — barrel exports

See the `admin-crud-scaffold` skill for complete templates.

## Conventions

### File Naming
- **Extension:** `.jsx` (not `.js` for components)
- **Name:** PascalCase matching the component name (e.g., `ProductCard.jsx`)
- **Export:** `export default` at the bottom of the file

### Import Aliases

Always use path aliases from `jsconfig.json`:

| Alias | Path | Example |
|-------|------|---------|
| `@components` | `components/index` | `import { Button, Modal } from '@components'` |
| `@components/*` | `components/*` | `import { ProductCard } from '@components/Products'` |
| `@hooks` | `hooks/index` | `import { useQuery, useDisclosure } from '@hooks'` |
| `@lib` | `lib/index` | `import { toaster, classnames } from '@lib'` |
| `@functions` | `functions/index` | `import { formatDate, generateSlug } from '@functions'` |
| `@constants` | `constants/index` | `import { DOCUMENTS } from '@constants'` |
| `@data` | `data/index` | `import { articleColumns } from '@data'` |
| `@site.config` | `site.config` | `import config from '@site.config'` |

### Styling

Use **Tailwind CSS** with the project's theme tokens:

```jsx
{/* Use project theme colors */}
<div className="bg-primary text-white">           {/* Primary dark color */}
<div className="bg-accent text-gray-900">         {/* Accent gold color */}
<div className="text-muted">                      {/* Muted text */}
<div className="border-metal-200">                {/* Border color */}

{/* Use project font families */}
<h1 className="font-heading">                     {/* Heading font (Poppins) */}
<p className="font-body">                         {/* Body font (Inter) */}
```

### Existing UI Components

Before creating new components, check if the project already provides what you need. Import from `@components`:

`AnimatedTitle`, `AreYouSure`, `Bone` (skeleton), `BrandButton`, `Button`, `CallToAction`, `ContextMenu`, `Embed`, `EmailMask`, `ErrorBoundary`, `ErrorFallback`, `FilterModal`, `FormattedTime`, `Image`, `Layout`, `Link`, `Loading`, `LoadingBubbles`, `Logo`, `MediaUpload`, `Modal`, `NoIndex`, `NoSsr`, `OpenGraph`, `Overflow`, `Pagination`, `Percent`, `Pill`, `Plural`, `PresentationLayout`, `Price`, `ProductSlider`, `Profile`, `SearchBar`, `SearchModal`, `ShowMore`, `Spinner`, `StatusCell`, `TikTokEmbed`, `Toaster`, `Tooltip`, `Trim`

### Hook Integration

Common hooks from `@hooks`:

| Hook | Use For |
|------|---------|
| `useQuery(url, options)` | Fetch data |
| `useInfiniteQuery(url, options)` | Paginated data |
| `useMutation(fn, options)` | Create/update/delete |
| `useDisclosure()` | Modal open/close state → `{ isOpen, show, hide, toggle }` |
| `useToggle(initial)` | Boolean toggle |
| `useDebounce(value, delay)` | Debounce search input |
| `useOnClickOutside(ref, handler)` | Close dropdown on outside click |
| `useIntersectionObserver(options)` | Lazy loading, scroll reveal |
| `useSwipeable(handlers)` | Touch swipe gestures |
| `useCollapsible()` | Accordion expand/collapse |
| `useSlugGenerator(sourceValue)` | Auto-generate URL slugs |

### File Length

Keep files short — **maximum 40-50 lines of code per file**. If a component grows beyond this, split it into smaller sub-components in the same feature directory.

### Status Components Pattern

When using `useQuery` or `useInfiniteQuery`, create separate components for each state:
- `{Entity}Loading.jsx` — use `Bone` component from `@components` for skeleton placeholders
- `{Entity}Error.jsx` — error message display
- `{Entity}Success.jsx` — actual content rendering

```jsx
// In the parent component
{status === 'loading' && <EntityLoading />}
{status === 'error' && <EntityError />}
{status === 'success' && <EntitySuccess data={data} />}
```

### Icons

Use Font Awesome icons in HTML format: `<i className="fas fa-bars"></i>`

### Image Component

For images, use the project's `Image` component with lazy-blur:

```jsx
import { Image } from '@components';
import { getImageUrl, getPlaceholderImageUrl } from '@functions';

<Image
  src={getImageUrl(images, 'medium')}
  placeholderSrc={getPlaceholderImageUrl(images)}
  effect="lazy-blur"
  alt="Description"
/>
```

#### If the Image Component Is Missing

If the project doesn't have the `Image` component yet (e.g., in a fresh starter), set it up:

1. **Install the dependency:**
   ```bash
   npm install react-lazy-load-image-component
   ```

2. **Create `components/Image.jsx`:**
   ```jsx
   import { classnames } from '@lib';
   import { LazyLoadImage } from 'react-lazy-load-image-component';
   import 'react-lazy-load-image-component/src/effects/black-and-white.css';
   import 'react-lazy-load-image-component/src/effects/opacity.css';

   const Image = ({ alt, src, srcSet, sizes, placeholderSrc, effect, className, wrapperClassName, ...rest }) => (
     <LazyLoadImage
       alt={alt}
       effect={effect}
       placeholderSrc={placeholderSrc}
       src={src}
       srcSet={srcSet}
       className={classnames('w-full h-full', className)}
       wrapperClassName={classnames('w-full h-full', wrapperClassName)}
       sizes={sizes}
       {...rest}
     />
   );

   export default Image;
   ```

3. **Create `css/lazy-loading-blur-effect.css`:**
   ```css
   .lazy-load-image-background.lazy-blur {
     filter: blur(15px);
   }
   .lazy-load-image-background.lazy-blur.lazy-load-image-loaded {
     filter: blur(0);
     transition: filter .3s;
   }
   ```

4. **Import it in `css/index.css`:**
   ```css
   @import 'lazy-loading-blur-effect.css';
   ```

5. **Create `functions/image-utils.js`:**
   ```js
   export const getImageUrl = (images, preferredSize = 'medium') => {
     const sizes = [preferredSize, 'medium', 'large', 'small', 'original'];
     for (const size of sizes) {
       if (images?.[size]?.path) return images[size].path;
     }
     return '';
   };

   export const getPlaceholderImageUrl = (images) => images?.small?.path || '';

   export const createImageSrcSet = (images) => {
     const set = {};
     if (images?.small?.path) set['480w'] = images.small.path;
     if (images?.medium?.path) set['768w'] = images.medium.path;
     if (images?.large?.path) set['1024w'] = images.large.path;
     if (images?.original?.path) set['1440w'] = images.original.path;
     return set;
   };
   ```

6. **Add barrel exports** in `components/index.js` and `functions/index.js`.
