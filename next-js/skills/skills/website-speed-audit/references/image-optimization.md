# Image Optimization Reference

## ffmpeg Commands for Image Processing

### Generate Tiny Placeholder (blur-up technique)

The blur-up technique shows a tiny, blurred version of the image while the full image loads. The project's `Image` component with `effect="lazy-blur"` handles the CSS transition.

```bash
# Tiny JPEG placeholder (~1-2KB, 40px wide)
ffmpeg -i input.jpg -vf "scale=40:-2" -q:v 10 output-placeholder.jpg

# Even tinier for inline base64 (~500B, 20px wide)
ffmpeg -i input.jpg -vf "scale=20:-2" -q:v 15 output-placeholder-tiny.jpg
```

### Convert to WebP

```bash
# High quality WebP (for hero images, ~80% smaller than JPEG)
ffmpeg -i input.jpg -quality 85 output.webp

# Standard quality WebP (for general images)
ffmpeg -i input.jpg -quality 80 output.webp

# Low quality WebP (for thumbnails)
ffmpeg -i input.jpg -quality 60 -vf "scale=400:-2" output-thumb.webp
```

### Resize Images

```bash
# Resize to max 1920px wide (maintain aspect ratio)
ffmpeg -i input.jpg -vf "scale='min(1920,iw)':-2" output-1920.jpg

# Resize to specific widths for srcSet
ffmpeg -i input.jpg -vf "scale=480:-2" output-480w.jpg
ffmpeg -i input.jpg -vf "scale=768:-2" output-768w.jpg
ffmpeg -i input.jpg -vf "scale=1024:-2" output-1024w.jpg
ffmpeg -i input.jpg -vf "scale=1440:-2" output-1440w.jpg
```

### Batch Processing Script

```bash
#!/bin/bash
# optimize-images.sh — Process all images in a directory
# Usage: ./optimize-images.sh /path/to/images

INPUT_DIR="${1:-.}"
OUTPUT_DIR="${INPUT_DIR}/optimized"
mkdir -p "$OUTPUT_DIR"

for file in "$INPUT_DIR"/*.{jpg,jpeg,png,JPG,JPEG,PNG}; do
  [ -f "$file" ] || continue
  filename=$(basename "$file" | sed 's/\.[^.]*$//')

  # Tiny placeholder for blur-up
  ffmpeg -i "$file" -vf "scale=40:-2" -q:v 10 "$OUTPUT_DIR/${filename}-placeholder.jpg" -y 2>/dev/null

  # WebP version (main display)
  ffmpeg -i "$file" -vf "scale='min(1920,iw)':-2" -quality 80 "$OUTPUT_DIR/${filename}.webp" -y 2>/dev/null

  # Responsive sizes
  ffmpeg -i "$file" -vf "scale='min(480,iw)':-2" -quality 75 "$OUTPUT_DIR/${filename}-480w.webp" -y 2>/dev/null
  ffmpeg -i "$file" -vf "scale='min(768,iw)':-2" -quality 75 "$OUTPUT_DIR/${filename}-768w.webp" -y 2>/dev/null
  ffmpeg -i "$file" -vf "scale='min(1024,iw)':-2" -quality 80 "$OUTPUT_DIR/${filename}-1024w.webp" -y 2>/dev/null

  echo "✓ $filename"
done

echo "Done! Output in $OUTPUT_DIR"
```

## Sharp (Node.js) Commands

### Installation

```bash
npm install sharp --save-dev
```

### Optimization Script

```js
// scripts/optimize-images.js
const sharp = require('sharp');
const fs = require('fs');
const path = require('path');

const inputDir = process.argv[2] || 'public/images';
const outputDir = path.join(inputDir, 'optimized');
fs.mkdirSync(outputDir, { recursive: true });

const extensions = ['.jpg', '.jpeg', '.png'];
const files = fs.readdirSync(inputDir).filter(f =>
  extensions.includes(path.extname(f).toLowerCase())
);

async function processImage(file) {
  const name = path.parse(file).name;
  const input = path.join(inputDir, file);

  // Tiny placeholder for blur-up (~1KB)
  await sharp(input)
    .resize(40)
    .jpeg({ quality: 20 })
    .toFile(path.join(outputDir, `${name}-placeholder.jpg`));

  // Full-size WebP
  await sharp(input)
    .resize(1920, null, { withoutEnlargement: true })
    .webp({ quality: 80 })
    .toFile(path.join(outputDir, `${name}.webp`));

  // Responsive sizes
  for (const [width, suffix] of [[480, '480w'], [768, '768w'], [1024, '1024w']]) {
    await sharp(input)
      .resize(width, null, { withoutEnlargement: true })
      .webp({ quality: 75 })
      .toFile(path.join(outputDir, `${name}-${suffix}.webp`));
  }

  console.log(`✓ ${name}`);
}

Promise.all(files.map(processImage)).then(() => console.log('Done!'));
```

Run with: `node scripts/optimize-images.js public/images`

## Project Integration

### Using with the Image Component

The project's `Image` component (from `@components`) wraps `react-lazy-load-image-component`:

```jsx
import { Image } from '@components';

// For backend images (multi-size objects)
import { getImageUrl, getPlaceholderImageUrl, createImageSrcSet } from '@functions';

<Image
  src={getImageUrl(images, 'medium')}
  placeholderSrc={getPlaceholderImageUrl(images)}
  srcSet={createImageSrcSet(images)}
  sizes="(max-width: 480px) 100vw, (max-width: 768px) 50vw, 33vw"
  effect="lazy-blur"
  alt="Description"
/>

// For static/optimized images
<Image
  src="/images/optimized/hero.webp"
  placeholderSrc="/images/optimized/hero-placeholder.jpg"
  effect="lazy-blur"
  alt="Description"
/>
```

### The Blur CSS

The blur transition is in `css/lazy-loading-blur-effect.css`:

```css
.lazy-load-image-background.lazy-blur {
  filter: blur(15px);
}
.lazy-load-image-background.lazy-blur.lazy-load-image-loaded {
  filter: blur(0);
  transition: filter .3s;
}
```

### Image Utilities (functions/image-utils.js)

| Function | Purpose |
|----------|---------|
| `getImageUrl(images, size)` | Get best URL for a size ('small', 'medium', 'large', 'original') with fallback |
| `getPlaceholderImageUrl(images)` | Get smallest image for blur placeholder |
| `createImageSrcSet(images)` | Generate srcSet object: 480w→small, 768w→medium, 1024w→large, 1440w→original |
| `getImageUrls(images)` | Extract all URL paths from multi-size object |
