# Lighthouse & Core Web Vitals Reference

## Core Web Vitals Thresholds

| Metric | Good | Needs Improvement | Poor |
|--------|------|-------------------|------|
| **LCP** (Largest Contentful Paint) | ≤2.5s | 2.5s–4.0s | >4.0s |
| **INP** (Interaction to Next Paint) | ≤200ms | 200ms–500ms | >500ms |
| **CLS** (Cumulative Layout Shift) | ≤0.1 | 0.1–0.25 | >0.25 |

## Other Key Metrics

| Metric | Good | Needs Improvement | Poor |
|--------|------|-------------------|------|
| **FCP** (First Contentful Paint) | ≤1.8s | 1.8s–3.0s | >3.0s |
| **TTFB** (Time to First Byte) | ≤800ms | 800ms–1800ms | >1800ms |
| **Speed Index** | ≤3.4s | 3.4s–5.8s | >5.8s |
| **TBT** (Total Blocking Time) | ≤200ms | 200ms–600ms | >600ms |

## Running Lighthouse

### CLI

```bash
# Full report as JSON
npx lighthouse http://localhost:3000 \
  --output=json \
  --output-path=/tmp/lighthouse-report.json \
  --chrome-flags="--headless --no-sandbox"

# Only performance category
npx lighthouse http://localhost:3000 \
  --only-categories=performance \
  --output=json \
  --output-path=/tmp/lighthouse-perf.json \
  --chrome-flags="--headless --no-sandbox"

# HTML report
npx lighthouse http://localhost:3000 \
  --output=html \
  --output-path=/tmp/lighthouse-report.html \
  --chrome-flags="--headless --no-sandbox"

# Mobile emulation (default) vs Desktop
npx lighthouse http://localhost:3000 --preset=desktop --output=json ...
```

### Parsing JSON Results

Key paths in the JSON output:

```
.categories.performance.score           → Overall score (0-1, multiply by 100)
.audits['largest-contentful-paint'].numericValue  → LCP in ms
.audits['cumulative-layout-shift'].numericValue   → CLS score
.audits['interaction-to-next-paint'].numericValue → INP in ms
.audits['first-contentful-paint'].numericValue    → FCP in ms
.audits['speed-index'].numericValue               → Speed Index in ms
.audits['total-blocking-time'].numericValue       → TBT in ms
.audits['server-response-time'].numericValue      → TTFB in ms
```

## Common Performance Issues & Fixes

### LCP (Largest Contentful Paint) > 2.5s

**Common causes in Next.js:**
1. **Unoptimized hero image** — Use WebP, add `preload` hint, use responsive sizes
2. **Slow server response** — Check `getServerSideProps` complexity, add caching
3. **Render-blocking CSS** — Minimize CSS, inline critical CSS
4. **Large JavaScript bundle** — Use `next/dynamic` for heavy components

**Fixes:**
```jsx
// Preload hero image
<link rel="preload" as="image" href="/images/hero.webp" type="image/webp" />

// Use responsive images with priority
<Image src={heroUrl} placeholderSrc={placeholderUrl} effect="lazy-blur" />
```

### CLS (Cumulative Layout Shift) > 0.1

**Common causes:**
1. **Images without dimensions** — Always set width/height or use aspect-ratio CSS
2. **Dynamic content insertion** — Ads, embeds, or lazy-loaded content pushing layout
3. **Font swap causing text reflow** — Use `font-display: swap` with matched fallback
4. **Client-side rendering changing layout** — Content that changes after hydration

**Fixes:**
```css
/* Reserve space for images */
.image-container { aspect-ratio: 16 / 9; }

/* Reserve space for embeds */
.embed-container { min-height: 400px; }
```

### INP (Interaction to Next Paint) > 200ms

**Common causes:**
1. **Heavy JavaScript on main thread** — Long tasks blocking interaction
2. **Expensive re-renders** — Unoptimized React components
3. **Synchronous operations** — Large data processing on click

**Fixes:**
- Use `React.memo` for components that re-render unnecessarily
- Use `useMemo`/`useCallback` for expensive computations
- Debounce input handlers
- Use `next/dynamic` for non-critical interactive components

### Render-Blocking Resources

**Check for:**
- CSS files loaded in `<head>` without `media` attribute
- JavaScript files without `async` or `defer`
- Google Fonts loaded synchronously

**Fixes:**
```html
<!-- Preconnect to font CDN -->
<link rel="preconnect" href="https://fonts.googleapis.com" />
<link rel="preconnect" href="https://fonts.gstatic.com" crossOrigin="anonymous" />

<!-- Load fonts with display=swap -->
<link href="https://fonts.googleapis.com/css2?family=Poppins:wght@400;500;600;700&display=swap" rel="stylesheet" />
```

### Large JavaScript Bundles

**Check with:**
```bash
# Analyze Next.js bundle
npx @next/bundle-analyzer
```

**Common heavy dependencies to dynamically import:**
- `react-quill` (~200KB) — only needed on admin pages with rich text
- `leaflet` + `react-leaflet` (~150KB) — only needed on map pages
- `framer-motion` (~100KB) — only needed on animated pages
- `react-calendar` (~50KB) — only needed on pages with calendars

```jsx
import dynamic from 'next/dynamic';

const RichTextEditor = dynamic(() => import('@components/Fields/RichText'), {
  ssr: false,
  loading: () => <div className="h-40 bg-gray-100 animate-pulse rounded" />,
});
```

### Unoptimized Images

**Audit checklist:**
- [ ] All dynamic images use `Image` component with `effect="lazy-blur"`
- [ ] All dynamic images have `placeholderSrc` for blur-up
- [ ] All dynamic images have `srcSet` for responsive loading
- [ ] Static images in `public/` are WebP format
- [ ] Static images have placeholder versions for blur-up
- [ ] No image is served larger than its display size
- [ ] Hero/above-fold images have preload hints

### Unoptimized Videos

**Audit checklist:**
- [ ] All videos have `preload="metadata"` (not `preload="auto"`)
- [ ] All videos have `muted` and `playsInline` attributes
- [ ] Background videos have poster frames
- [ ] Videos are compressed (H.264, CRF ≥26)
- [ ] Multiple quality versions exist for responsive delivery
- [ ] Below-fold videos use Intersection Observer for play/pause
- [ ] Video files use `faststart` flag for progressive loading
