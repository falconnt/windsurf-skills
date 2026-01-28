---
description: Use when converting SVG vector graphics to PNG raster images at specific sizes
---

# SVG to PNG Conversion

## Overview

Convert SVG files to PNG images at specified dimensions using `svgexport`.

## Prerequisites

```bash
npm install -g svgexport
```

**Note:** After switching Node versions with `nvm use`, reinstall the package.

## Basic Usage

```bash
svgexport input.svg output.png [width]:[height]
```

### Examples

```bash
# Export at 256x256
svgexport icon.svg icon-256.png 256:256

# Export at 64x64
svgexport icon.svg icon-64.png 64:64

# Export at original size (uses viewBox dimensions)
svgexport icon.svg icon.png
```

## Batch Export for Icons

Generate all standard icon sizes:

```bash
svgexport icon.svg icon-256.png 256:256
svgexport icon.svg icon-128.png 128:128
svgexport icon.svg icon-64.png 64:64
svgexport icon.svg icon-48.png 48:48
svgexport icon.svg icon-32.png 32:32
svgexport icon.svg icon-16.png 16:16
```

## Advanced Options

```bash
# Export with quality setting (for JPEG)
svgexport input.svg output.jpg 256:256 80%

# Export specific viewBox region
svgexport input.svg output.png 256:256 0:0:100:100

# Export with padding
svgexport input.svg output.png pad 256:256
```

## Output Format

The output format is determined by the file extension:
- `.png` - PNG (recommended for icons)
- `.jpg` / `.jpeg` - JPEG
- `.pdf` - PDF

## Troubleshooting

| Issue | Solution |
|-------|----------|
| `svgexport: command not found` | Run `npm install -g svgexport` |
| Command not found after nvm switch | Reinstall with current Node version |
| Output is blank/wrong size | Check SVG has valid `viewBox` attribute |
| Gradients not rendering | svgexport uses Puppeteer; complex SVGs may need simplification |

## How It Works

svgexport uses Puppeteer (headless Chrome) to render the SVG, then captures a screenshot at the specified dimensions. This ensures accurate rendering of CSS, gradients, and animations.
