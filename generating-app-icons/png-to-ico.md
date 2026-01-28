---
description: Use when converting PNG images to ICO format for Windows application icons
---

# PNG to ICO Conversion

## Overview

Convert PNG images to Windows ICO format. ICO files can contain multiple resolutions for proper display at different sizes.

## Prerequisites

```bash
npm install png-to-ico
```

**Important:** Install locally in your project, not globally. The CLI has issues with binary output in PowerShell.

## The Problem with CLI

The `png-to-ico` CLI outputs binary to stdout:

```bash
# DON'T DO THIS - PowerShell corrupts binary output
png-to-ico icon.png > output.ico  # CORRUPTED FILE
```

PowerShell treats the output as text and corrupts the binary data.

## Solution: Use Node.js API

Create a script to write the ICO file properly:

```javascript
// create-ico.js
const fs = require('fs');
const { default: pngToIco } = require('png-to-ico');

async function createIco(pngFiles, outputPath) {
    const icoBuffer = await pngToIco(pngFiles);
    fs.writeFileSync(outputPath, icoBuffer);
    console.log(`Created: ${outputPath} (${icoBuffer.length} bytes)`);
}

// Include multiple sizes for best results
const pngFiles = [
    'icon-256.png',
    'icon-64.png',
    'icon-32.png',
    'icon-16.png'
];

createIco(pngFiles, 'AppIcon.ico').catch(console.error);
```

Run with:

```bash
node create-ico.js
```

## Recommended Icon Sizes

| Size | Usage |
|------|-------|
| 256x256 | High-DPI displays, large icons |
| 64x64 | Medium icons |
| 48x48 | Taskbar (optional) |
| 32x32 | Standard icons |
| 16x16 | Small icons, system tray |

**Minimum:** 16, 32, and 256 for good coverage.

## Verifying the ICO File

Check the file was created correctly:

```powershell
Get-Item AppIcon.ico | Select-Object Name, Length
```

A valid multi-size ICO should be 50-300KB depending on included sizes.

## Troubleshooting

| Issue | Cause | Solution |
|-------|-------|----------|
| "Argument must be a picture that can be used as Icon" | Corrupted ICO | Use Node.js API, not shell redirect |
| `pngToIco is not a function` | ESM module export | Use `{ default: pngToIco }` destructuring |
| File is 0 bytes | Write failed | Check file path and permissions |
| Very large file (300KB+) but invalid | Text encoding corruption | Use `fs.writeFileSync` with Buffer |

## Node.js Version Requirements

- **Node.js 20+** required for `png-to-ico@3.x`
- After `nvm use` switch, reinstall the package

## Alternative: cmd.exe Redirect

If you must use CLI, use cmd.exe instead of PowerShell:

```bash
cmd /c "png-to-ico icon.png > output.ico"
```

This avoids PowerShell's text encoding, but the Node.js API is more reliable.
