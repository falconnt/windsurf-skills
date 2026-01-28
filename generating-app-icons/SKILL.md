---
description: Use when creating application icons for Windows apps, from initial SVG design through to final ICO file
---

# Generating App Icons

## Overview

Create application icons from scratch: design SVG concepts, select the best option, convert to multi-resolution ICO for Windows applications.

## Pipeline

1. **Generate SVG designs** - Create 3-4 icon concepts
2. **Choose the design** - Preview at multiple sizes, select winner
3. **Convert to PNG** - Export at required resolutions (256, 64, 32, 16)
4. **Convert to ICO** - Combine PNGs into Windows ICO file

## Prerequisites

- **Node.js 20+** (required for png-to-ico package)
- **npm** for package installation

## Step 1: Generate SVG Designs

Create 3-4 icon concepts as SVG files. Consider the app's purpose and brand.

### Icon Design Approaches

| Style | Best For | Example Elements |
|-------|----------|------------------|
| **Symbolic** | Developer tools, utilities | Chevrons, brackets, geometric shapes |
| **Metaphorical** | Search, discovery apps | Magnifying glass, lens, compass |
| **Abstract** | Universal concepts | Infinity symbol, orbs, interconnected shapes |
| **Letterform** | Brand-focused | Stylized first letter, monogram |

### SVG Template

```xml
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 256 256">
  <defs>
    <linearGradient id="gradient" x1="0%" y1="0%" x2="100%" y2="100%">
      <stop offset="0%" style="stop-color:#14B8A6"/>
      <stop offset="100%" style="stop-color:#0D9488"/>
    </linearGradient>
  </defs>
  
  <!-- Background -->
  <rect x="18" y="18" width="220" height="220" rx="44" fill="#0F172A"/>
  
  <!-- Your icon elements here -->
  
</svg>
```

### Design Guidelines

- **ViewBox:** Use 256x256 for best scaling
- **Padding:** Keep 18-24px margin from edges
- **Corner radius:** 44px for rounded square backgrounds (matches Windows 11)
- **Colors:** Use app's accent color; avoid too many colors
- **Simplicity:** Details disappear at 16x16; test small sizes early

### Save Multiple Options

```
docs/plans/image/icon-designs/
├── option-1-symbolic.svg
├── option-2-metaphor.svg
├── option-3-abstract.svg
└── option-4-letterform.svg
```

## Step 2: Choose the Design

Create an HTML preview to compare options at all sizes:

```html
<!DOCTYPE html>
<html>
<head>
    <title>Icon Preview</title>
    <style>
        body { background: #1a1a2e; color: white; font-family: system-ui; }
        .preview { display: flex; gap: 20px; align-items: end; margin: 20px; }
        .size-label { text-align: center; opacity: 0.7; margin-top: 8px; }
    </style>
</head>
<body>
    <h2>Option 1: Symbolic</h2>
    <div class="preview">
        <div><img src="option-1.svg" width="256"><div class="size-label">256</div></div>
        <div><img src="option-1.svg" width="64"><div class="size-label">64</div></div>
        <div><img src="option-1.svg" width="32"><div class="size-label">32</div></div>
        <div><img src="option-1.svg" width="16"><div class="size-label">16</div></div>
    </div>
    <!-- Repeat for other options -->
</body>
</html>
```

Open in browser, evaluate at small sizes, select the winner.

## Step 3: Install Tools & Convert to PNG

```bash
npm install -g svgexport
npm install png-to-ico  # Install locally, not globally
```

**Why local png-to-ico?** The CLI uses stdout for binary output, which PowerShell corrupts. Use Node.js API instead.

Export the chosen SVG to multiple PNG sizes:

```bash
svgexport chosen-icon.svg icon-256.png 256:256
svgexport chosen-icon.svg icon-64.png 64:64
svgexport chosen-icon.svg icon-32.png 32:32
svgexport chosen-icon.svg icon-16.png 16:16
```

## Step 4: Convert PNGs to ICO

**Critical:** Do NOT use shell redirection (`>`) for binary output in PowerShell—it corrupts the file.

Create a Node.js script to write the ICO properly:

```javascript
const fs = require('fs');
const path = require('path');
const { default: pngToIco } = require('png-to-ico');

async function createIco(pngFiles, outputPath) {
    const icoBuffer = await pngToIco(pngFiles);
    fs.writeFileSync(outputPath, icoBuffer);
    console.log(`Created: ${outputPath} (${icoBuffer.length} bytes)`);
}

const pngFiles = [
    'icon-256.png',
    'icon-64.png', 
    'icon-32.png',
    'icon-16.png'
];

createIco(pngFiles, 'AppIcon.ico');
```

Run with: `node create-ico.js`

## Red Flags

| Symptom | Cause | Fix |
|---------|-------|-----|
| "Argument must be a picture that can be used as Icon" | Corrupted ICO file | Use Node.js API, not shell redirect |
| ICO file very large (300KB+) but invalid | PowerShell text encoding | Use `fs.writeFileSync` with Buffer |
| Icon not updating after rebuild | App still running | Close app before rebuild |
| svgexport not found after nvm switch | Package in old Node version | Reinstall with current Node |

## Quick Reference

```bash
# Full pipeline
npm install -g svgexport
npm install png-to-ico

svgexport design.svg icon-256.png 256:256
svgexport design.svg icon-64.png 64:64
svgexport design.svg icon-32.png 32:32
svgexport design.svg icon-16.png 16:16

node create-ico.js
```

## SVG Design Tips for Icons

- Use 256x256 viewBox for best scaling
- Avoid fine details that disappear at 16x16
- Test at all sizes before finalizing
- Use solid colors; gradients may not render well at small sizes
- Keep padding inside the viewBox (icons look better with breathing room)
