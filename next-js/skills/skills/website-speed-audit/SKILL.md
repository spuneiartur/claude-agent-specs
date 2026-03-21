---
name: website-speed-audit
description: >
  Comprehensive website performance audit and optimization skill. Identifies and automatically
  fixes performance issues including image optimization, video compression, lazy loading, Core
  Web Vitals, bundle size, and rendering strategy. Uses Lighthouse (via CLI or MCP when available),
  ffmpeg for media processing, and the project's existing Image component with blur-up lazy loading.
  Use this skill whenever the user mentions: website speed, page load time, performance audit,
  Core Web Vitals, Lighthouse, optimize images, compress videos, lazy loading, LCP, CLS, FID, INP,
  slow website, speed up, performance optimization, image compression, video optimization, blur
  placeholder, WebP conversion, media audit, bundle size, or wants to improve their website's
  loading performance. Also trigger when the user says "my site is slow", "optimize for speed",
  "reduce load time", "improve performance", or asks about image/video optimization in any context.
---

# Website Speed Audit & Optimization

A three-phase skill: **Audit** (identify issues) → **Fix** (automated optimizations) → **Report** (document changes). Designed for Next.js Pages Router projects with the starter's Image component and lazy-blur system.

---

## Phase 1: Audit

### Step 1: Lighthouse Audit (preferred)

Check if Lighthouse CLI or an MCP tool is available:

```bash
# Check for Lighthouse CLI
npx lighthouse --version 2>/dev/null || echo "Lighthouse not available"
```

If available, run:
```bash
npx lighthouse http://localhost:3000 --output=json --output-path=/tmp/lighthouse-report.json --chrome-flags="--headless --no-sandbox"
```

Parse the JSON for Core Web Vitals: LCP, CLS, INP, FCP, TTFB, Speed Index. Read `references/lighthouse-checks.md` for thresholds and interpretation.

If Lighthouse is not available, proceed with manual codebase audit.

### Step 2: Manual Codebase Audit

Regardless of whether Lighthouse ran, audit the codebase for these categories:

#### Image Audit

Scan all components for image usage:

1. **Find images without lazy loading**: Search for `<img` tags or `<Image` components missing `effect="lazy-blur"`. Every image loaded from the backend (multi-size objects) should use the project's `Image` component with blur-up.

2. **Find images without placeholders**: Search for `Image` components missing `placeholderSrc`. Every dynamic image should use `getPlaceholderImageUrl(images)` from `@functions`.

3. **Find images without srcSet**: Search for `Image` components missing responsive `srcSet`. Dynamic images should use `createImageSrcSet(images)` from `@functions`.

4. **Check static images in public/**: List files in `public/images/` and `public/` — identify large files (>200KB) that could be compressed or converted to WebP.

5. **Check image formats**: Flag JPEG/PNG files that should be WebP for better compression.

#### Video Audit

1. **Check video file sizes**: List `public/videos/` and flag files over 5MB.
2. **Check video attributes**: Find `<video>` tags — verify they have `preload="metadata"` (not `preload="auto"`), `muted`, `playsInline`.
3. **Check for poster images**: Videos should have a `poster` attribute for immediate visual display before load.
4. **Check play behavior**: Background videos should only play when visible (Intersection Observer).

#### Font Audit

1. **Check font-display**: Verify Google Fonts links include `&display=swap`.
2. **Check preconnect**: Verify `<link rel="preconnect" href="https://fonts.googleapis.com">` exists in `_document.js`.
3. **Count font weights**: Flag if more than 4 font weights are loaded (each adds ~20KB).

#### Bundle & Rendering Audit

1. **Check large dependencies**: Look at `package.json` for heavy packages that could be dynamically imported (e.g., `react-quill`, `leaflet`, `framer-motion` on pages that don't use them).
2. **Suggest next/dynamic**: Components that use heavy libraries should be dynamically imported with `ssr: false` where appropriate.
3. **Check CSS imports**: Count imports in `css/index.css` — flag unused stylesheets.
4. **Check rendering strategy**: Pages using `getServerSideProps` when they could use `getStaticProps` with ISR.

#### Third-Party Script Audit

1. **Check for blocking scripts**: Look for `<script>` tags without `async` or `defer`.
2. **Check for unnecessary scripts**: Flag analytics, chat widgets, etc. that load on every page.

### Step 3: Present Audit Findings

Present findings grouped by impact:
- **Critical** (>1s impact): Unoptimized hero images/videos, render-blocking resources, huge bundles
- **High** (0.3-1s): Missing lazy loading, no WebP, uncompressed videos
- **Medium** (<0.3s): Missing srcSet, font optimization, unnecessary scripts
- **Low**: Minor CSS cleanup, optional preloading improvements

---

## Phase 2: Automated Fixes

### Image Optimization

#### Fix 1: Enforce Lazy Loading on Dynamic Images

For every `Image` component using backend images (multi-size objects), ensure it has the full optimization setup:

```jsx
import { Image } from '@components';
import { getImageUrl, getPlaceholderImageUrl, createImageSrcSet } from '@functions';

<Image
  src={getImageUrl(images, 'medium')}
  placeholderSrc={getPlaceholderImageUrl(images)}
  srcSet={createImageSrcSet(images)}
  sizes="(max-width: 480px) 100vw, (max-width: 768px) 50vw, 33vw"
  effect="lazy-blur"
  alt="Descriptive alt text"
/>
```

The blur-up effect is handled by `css/lazy-loading-blur-effect.css`:
- Shows blurred placeholder (15px blur) while loading
- Transitions to sharp image with 0.3s filter animation

#### Fix 2: Static Image Dual-Version Generation

For images in `public/` that don't have pre-generated sizes, create two versions using ffmpeg:

```bash
#!/bin/bash
# scripts/optimize-images.sh
# Usage: ./scripts/optimize-images.sh public/images/

INPUT_DIR="${1:-.}"
OUTPUT_DIR="${INPUT_DIR}/optimized"
mkdir -p "$OUTPUT_DIR"

for file in "$INPUT_DIR"/*.{jpg,jpeg,png,JPG,JPEG,PNG}; do
  [ -f "$file" ] || continue
  filename=$(basename "$file" | sed 's/\.[^.]*$//')

  # Generate tiny placeholder (40px wide, heavy JPEG compression, ~1-2KB)
  ffmpeg -i "$file" -vf "scale=40:-2" -q:v 10 "$OUTPUT_DIR/${filename}-placeholder.jpg" -y 2>/dev/null

  # Generate optimized WebP version
  ffmpeg -i "$file" -vf "scale='min(1920,iw)':-2" -quality 80 "$OUTPUT_DIR/${filename}.webp" -y 2>/dev/null

  echo "Optimized: $filename"
done

echo "Done! Optimized images in $OUTPUT_DIR"
```

Then update components to use the optimized versions:

```jsx
<Image
  src="/images/optimized/hero.webp"
  placeholderSrc="/images/optimized/hero-placeholder.jpg"
  effect="lazy-blur"
  alt="Hero image"
/>
```

#### Fix 3: Sharp-Based Optimization (alternative to ffmpeg)

If `sharp` is available (Node.js), generate optimized versions programmatically:

```bash
# Install sharp
npm install sharp --save-dev
```

```js
// scripts/optimize-images.js
const sharp = require('sharp');
const fs = require('fs');
const path = require('path');

const inputDir = process.argv[2] || 'public/images';
const outputDir = path.join(inputDir, 'optimized');
fs.mkdirSync(outputDir, { recursive: true });

const files = fs.readdirSync(inputDir).filter(f => /\.(jpg|jpeg|png)$/i.test(f));

for (const file of files) {
  const name = path.parse(file).name;
  const input = path.join(inputDir, file);

  // Tiny placeholder (~1KB)
  sharp(input).resize(40).jpeg({ quality: 20 }).toFile(path.join(outputDir, `${name}-placeholder.jpg`));

  // Optimized WebP
  sharp(input).resize(1920, null, { withoutEnlargement: true }).webp({ quality: 80 }).toFile(path.join(outputDir, `${name}.webp`));

  console.log(`Optimized: ${name}`);
}
```

### Video Optimization

#### Fix 4: Compress Videos with ffmpeg

```bash
# Web-optimized MP4 (H.264, CRF 28, faststart for streaming)
ffmpeg -i input.mp4 -vcodec libx264 -crf 28 -preset slow -movflags +faststart -an output-compressed.mp4

# Create 720p version for mobile
ffmpeg -i input.mp4 -vcodec libx264 -crf 28 -preset slow -movflags +faststart -vf "scale=-2:720" -an output-720p.mp4

# Create 1080p version for desktop
ffmpeg -i input.mp4 -vcodec libx264 -crf 26 -preset slow -movflags +faststart -vf "scale=-2:1080" -an output-1080p.mp4

# Generate poster frame (first frame as JPEG)
ffmpeg -i input.mp4 -vframes 1 -q:v 2 poster.jpg

# Generate poster frame as WebP
ffmpeg -i input.mp4 -vframes 1 -quality 80 poster.webp
```

Read `references/video-optimization.md` for the full command reference.

#### Fix 5: Scroll-Triggered Video Playback

For hero/background videos, implement play-only-when-visible:

```jsx
import { useEffect, useRef, useState } from 'react';

const VideoHero = ({ src, poster }) => {
  const videoRef = useRef(null);
  const [isLoaded, setIsLoaded] = useState(false);

  useEffect(() => {
    const video = videoRef.current;
    if (!video) return;

    const observer = new IntersectionObserver(
      ([entry]) => {
        if (entry.isIntersecting) {
          video.play().catch(() => {});
        } else {
          video.pause();
        }
      },
      { threshold: 0.25 }
    );

    observer.observe(video);
    return () => observer.disconnect();
  }, []);

  return (
    <div className="relative w-full h-screen overflow-hidden">
      {/* Poster image shown until video loads */}
      {!isLoaded && poster && (
        <img src={poster} alt="" className="absolute inset-0 w-full h-full object-cover" />
      )}
      <video
        ref={videoRef}
        src={src}
        poster={poster}
        muted
        loop
        playsInline
        preload="metadata"
        onLoadedData={() => setIsLoaded(true)}
        className={`absolute inset-0 w-full h-full object-cover transition-opacity duration-700 ${isLoaded ? 'opacity-100' : 'opacity-0'}`}
      />
    </div>
  );
};
```

This pattern is already used in the project's `ContactHero.jsx` — reference it.

#### Fix 6: Video Preloading Strategy

For above-fold videos, add preload hints in `_document.js`:

```jsx
<link rel="preload" as="video" type="video/mp4" href="/videos/hero-compressed.mp4" />
```

For below-fold videos, use `preload="none"` to prevent any data transfer until the user scrolls near them.

### General Performance Fixes

#### Fix 7: Dynamic Imports for Heavy Components

```jsx
import dynamic from 'next/dynamic';

// Only load RichText editor when needed
const RichText = dynamic(() => import('@components/Fields/RichText'), { ssr: false });

// Only load map component when needed
const Map = dynamic(() => import('@components/Map'), {
  ssr: false,
  loading: () => <div className="h-64 bg-gray-100 animate-pulse rounded-lg" />,
});
```

#### Fix 8: Font Optimization

Ensure `_document.js` has proper font loading:

```jsx
<link rel="preconnect" href="https://fonts.googleapis.com" />
<link rel="preconnect" href="https://fonts.gstatic.com" crossOrigin="anonymous" />
<link href="https://fonts.googleapis.com/css2?family=Poppins:wght@400;500;600;700&family=Inter:wght@400;500;600&display=swap" rel="stylesheet" />
```

Verify the CSS includes `font-display: swap` (Google Fonts adds this via the `&display=swap` parameter).

---

## Phase 3: Report

Generate a summary of all findings and fixes. List:

1. **Audit results** — what was found, categorized by severity
2. **Automated fixes applied** — what files were changed and how
3. **Manual actions needed** — things that require user intervention (e.g., re-exporting images from source, backend changes for image sizes)
4. **Before/after comparison** — if Lighthouse was available, show score changes

Present inline in the conversation — no need for an HTML report unless the user requests one.

---

## Reference Files

Read these for detailed command references and patterns:
- `references/image-optimization.md` — ffmpeg and sharp commands, blur-up implementation details
- `references/video-optimization.md` — ffmpeg commands for all video scenarios, scroll-play patterns
- `references/lighthouse-checks.md` — Core Web Vitals thresholds, scoring, common fixes

## Key Project Files

These files are central to the project's existing performance infrastructure:
- `components/Image.jsx` — Custom lazy-blur Image component
- `functions/image-utils.js` — `getImageUrl()`, `getPlaceholderImageUrl()`, `createImageSrcSet()`
- `css/lazy-loading-blur-effect.css` — Blur transition CSS (15px blur → 0 with 0.3s transition)
- `components/Contact/ContactHero.jsx` — Reference implementation of video with poster fade-in
- `next.config.js` — Next.js configuration
- `_document.js` — Font loading, preload hints

## Setting Up the Image Infrastructure (If Missing)

If the project doesn't have the Image component and lazy-blur system yet (e.g., fresh starter), set it up before optimizing:

1. **Install:** `npm install react-lazy-load-image-component`

2. **Create `components/Image.jsx`:**
```jsx
import { classnames } from '@lib';
import { LazyLoadImage } from 'react-lazy-load-image-component';
import 'react-lazy-load-image-component/src/effects/black-and-white.css';
import 'react-lazy-load-image-component/src/effects/opacity.css';

const Image = ({ alt, src, srcSet, sizes, placeholderSrc, effect, className, wrapperClassName, ...rest }) => (
  <LazyLoadImage
    alt={alt} effect={effect} placeholderSrc={placeholderSrc} src={src} srcSet={srcSet}
    className={classnames('w-full h-full', className)}
    wrapperClassName={classnames('w-full h-full', wrapperClassName)}
    sizes={sizes} {...rest}
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

4. **Add `@import 'lazy-loading-blur-effect.css';`** to `css/index.css`

5. **Create `functions/image-utils.js`** with `getImageUrl`, `getPlaceholderImageUrl`, `createImageSrcSet` — see the `component-factory` skill for the full implementation.

6. **Add barrel exports** in `components/index.js` and `functions/index.js`.
