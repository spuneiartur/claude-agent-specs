# Video Optimization Reference

## ffmpeg Commands

### Web-Optimized Compression

```bash
# Standard web compression (H.264, CRF 28, good balance of quality/size)
ffmpeg -i input.mp4 \
  -vcodec libx264 \
  -crf 28 \
  -preset slow \
  -movflags +faststart \
  -an \
  output-compressed.mp4

# Higher quality (CRF 23, for hero sections)
ffmpeg -i input.mp4 \
  -vcodec libx264 \
  -crf 23 \
  -preset slow \
  -movflags +faststart \
  -an \
  output-hq.mp4
```

Key flags:
- `-crf 28`: Quality level (18=near-lossless, 23=good, 28=acceptable, 32=low)
- `-preset slow`: Better compression ratio (use `medium` for faster encoding)
- `-movflags +faststart`: Moves metadata to start of file for streaming
- `-an`: Removes audio track (background videos don't need audio)

### Multiple Bitrate Versions

```bash
# 720p mobile version
ffmpeg -i input.mp4 \
  -vcodec libx264 -crf 28 -preset slow \
  -movflags +faststart \
  -vf "scale=-2:720" \
  -an \
  output-720p.mp4

# 1080p desktop version
ffmpeg -i input.mp4 \
  -vcodec libx264 -crf 26 -preset slow \
  -movflags +faststart \
  -vf "scale=-2:1080" \
  -an \
  output-1080p.mp4

# 4K version (if source is 4K)
ffmpeg -i input.mp4 \
  -vcodec libx264 -crf 24 -preset slow \
  -movflags +faststart \
  -vf "scale=-2:2160" \
  -an \
  output-4k.mp4
```

### Generate Poster Frame

```bash
# First frame as JPEG
ffmpeg -i input.mp4 -vframes 1 -q:v 2 poster.jpg

# First frame as WebP
ffmpeg -i input.mp4 -vframes 1 -quality 80 poster.webp

# Frame at specific timestamp (e.g., 2 seconds in)
ffmpeg -i input.mp4 -ss 00:00:02 -vframes 1 -q:v 2 poster.jpg

# Tiny poster placeholder for blur-up
ffmpeg -i input.mp4 -vframes 1 -vf "scale=40:-2" -q:v 10 poster-placeholder.jpg
```

### Video to GIF (for short loops, <5s)

```bash
# High quality GIF with palette optimization
ffmpeg -i input.mp4 -vf "fps=15,scale=480:-2:flags=lanczos,split[s0][s1];[s0]palettegen[p];[s1][p]paletteuse" output.gif
```

### Extract Frames (for scroll-driven animation)

```bash
# Extract at 30fps as JPEG
ffmpeg -i input.mp4 -vf "fps=30,scale=1920:-2" -q:v 5 frames/frame-%04d.jpg

# Extract at 24fps as WebP (smaller files)
ffmpeg -i input.mp4 -vf "fps=24,scale=1920:-2" -quality 60 frames/frame-%04d.webp
```

### Batch Compression Script

```bash
#!/bin/bash
# optimize-videos.sh — Compress all videos in a directory
# Usage: ./optimize-videos.sh public/videos

INPUT_DIR="${1:-public/videos}"
OUTPUT_DIR="${INPUT_DIR}/optimized"
mkdir -p "$OUTPUT_DIR"

for file in "$INPUT_DIR"/*.mp4; do
  [ -f "$file" ] || continue
  filename=$(basename "$file" .mp4)

  # Compressed version
  ffmpeg -i "$file" -vcodec libx264 -crf 28 -preset slow -movflags +faststart -an "$OUTPUT_DIR/${filename}-compressed.mp4" -y 2>/dev/null

  # 720p mobile version
  ffmpeg -i "$file" -vcodec libx264 -crf 28 -preset slow -movflags +faststart -vf "scale=-2:720" -an "$OUTPUT_DIR/${filename}-720p.mp4" -y 2>/dev/null

  # Poster frame
  ffmpeg -i "$file" -vframes 1 -q:v 2 "$OUTPUT_DIR/${filename}-poster.jpg" -y 2>/dev/null

  # Poster placeholder for blur-up
  ffmpeg -i "$file" -vframes 1 -vf "scale=40:-2" -q:v 10 "$OUTPUT_DIR/${filename}-poster-placeholder.jpg" -y 2>/dev/null

  original_size=$(du -h "$file" | cut -f1)
  compressed_size=$(du -h "$OUTPUT_DIR/${filename}-compressed.mp4" | cut -f1)
  echo "✓ $filename: $original_size → $compressed_size"
done

echo "Done! Output in $OUTPUT_DIR"
```

## Scroll-Triggered Video Playback Pattern

### Intersection Observer Play/Pause

Only play videos when they're visible in the viewport:

```jsx
import { useEffect, useRef, useState } from 'react';

const VideoSection = ({ src, poster, posterPlaceholder }) => {
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
    <div className="relative overflow-hidden">
      {/* Poster with blur-up shown until video loads */}
      {!isLoaded && (
        <Image
          src={poster}
          placeholderSrc={posterPlaceholder}
          effect="lazy-blur"
          alt=""
          className="absolute inset-0 w-full h-full object-cover"
        />
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
        className={`w-full h-full object-cover transition-opacity duration-700 ${isLoaded ? 'opacity-100' : 'opacity-0'}`}
      />
    </div>
  );
};
```

### Reference Implementation

The project's `ContactHero.jsx` demonstrates this pattern:
- Shows a placeholder image while video loads
- Uses `onLoadedData` event to fade in the video
- Video settings: `autoPlay muted loop playsInline`
- Gradient overlay for text readability

### Preloading Strategy

```jsx
// In _document.js for above-fold videos
<link rel="preload" as="video" type="video/mp4" href="/videos/hero-compressed.mp4" />

// For below-fold videos, use preload="none" on the video element
<video preload="none" ...>
```

## Responsive Video Sources

For serving different video qualities based on viewport:

```jsx
<video muted loop playsInline preload="metadata">
  <source src="/videos/hero-720p.mp4" type="video/mp4" media="(max-width: 768px)" />
  <source src="/videos/hero-1080p.mp4" type="video/mp4" />
</video>
```

Note: The `media` attribute on `<source>` has limited browser support. A more reliable approach is to detect viewport width in JS and set `video.src` dynamically.
