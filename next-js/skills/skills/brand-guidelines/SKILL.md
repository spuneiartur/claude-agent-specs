---
name: brand-guidelines
description: Applies Anthropic's official brand colors and typography to any sort of artifact that may benefit from having Anthropic's look-and-feel. Use it when brand colors or style guidelines, visual formatting, or company design standards apply.
license: Complete terms in LICENSE.txt
---

# Anthropic Brand Styling

## Overview

To access Anthropic's official brand identity and style resources, use this skill.

**Keywords**: branding, corporate identity, visual identity, post-processing, styling, brand colors, typography, Anthropic brand, visual formatting, visual design

## Brand Guidelines

### Colors

**Main Colors:**

- Dark: `#141413` - Primary text and dark backgrounds
- Light: `#faf9f5` - Light backgrounds and text on dark
- Mid Gray: `#b0aea5` - Secondary elements
- Light Gray: `#e8e6dc` - Subtle backgrounds

**Accent Colors:**

- Orange: `#d97757` - Primary accent
- Blue: `#6a9bcc` - Secondary accent
- Green: `#788c5d` - Tertiary accent

### Typography

- **Headings**: Poppins (with Arial fallback)
- **Body Text**: Lora (with Georgia fallback)
- **Note**: Fonts should be pre-installed in your environment for best results

## Features

### Smart Font Application

- Applies Poppins font to headings (24pt and larger)
- Applies Lora font to body text
- Automatically falls back to Arial/Georgia if custom fonts unavailable
- Preserves readability across all systems

### Text Styling

- Headings (24pt+): Poppins font
- Body text: Lora font
- Smart color selection based on background
- Preserves text hierarchy and formatting

### Shape and Accent Colors

- Non-text shapes use accent colors
- Cycles through orange, blue, and green accents
- Maintains visual interest while staying on-brand

## Technical Details

### Font Management

- Uses system-installed Poppins and Lora fonts when available
- Provides automatic fallback to Arial (headings) and Georgia (body)
- No font installation required - works with existing system fonts
- For best results, pre-install Poppins and Lora fonts in your environment

### Color Application

- Uses RGB color values for precise brand matching
- Applied via python-pptx's RGBColor class
- Maintains color fidelity across different systems

## Web / Next.js Application Branding

When applying brand guidelines to a web application or Next.js project, use these outputs instead of python-pptx:

### CSS Variables

```css
:root {
  --brand-dark: #141413;
  --brand-light: #faf9f5;
  --brand-mid-gray: #b0aea5;
  --brand-light-gray: #e8e6dc;
  --brand-orange: #d97757;
  --brand-blue: #6a9bcc;
  --brand-green: #788c5d;
  --font-heading: 'Poppins', Arial, sans-serif;
  --font-body: 'Lora', Georgia, serif;
}
```

### Tailwind CSS Configuration

Add to `tailwind.config.js` → `theme.extend`:

```js
colors: {
  primary: '#141413',
  light: '#faf9f5',
  muted: '#b0aea5',
  accent: '#d97757',
  'accent-blue': '#6a9bcc',
  'accent-green': '#788c5d',
  'metal-100': '#e8e6dc',
  'metal-200': '#b0aea5',
},
fontFamily: {
  heading: ['Poppins', 'Arial', 'sans-serif'],
  body: ['Lora', 'Georgia', 'serif'],
},
```

### Google Fonts Loading

Add to `_document.js` `<Head>`:

```html
<link rel="preconnect" href="https://fonts.googleapis.com" />
<link rel="preconnect" href="https://fonts.gstatic.com" crossOrigin="anonymous" />
<link href="https://fonts.googleapis.com/css2?family=Poppins:wght@400;500;600;700&family=Lora:wght@400;500;600;700&display=swap" rel="stylesheet" />
```

### Component-Level Application

```jsx
{/* Heading with brand font */}
<h1 className="font-heading text-primary text-4xl font-bold">Brand Heading</h1>

{/* Body text with brand font */}
<p className="font-body text-primary leading-relaxed">Body text content</p>

{/* Accent elements */}
<button className="bg-accent text-white px-6 py-3 rounded-lg font-heading font-semibold">
  Call to Action
</button>

{/* Light background section */}
<section className="bg-light py-16">
  <div className="container mx-auto px-4">
    {/* Content on light background */}
  </div>
</section>
```
