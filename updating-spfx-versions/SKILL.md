---
name: updating-spfx-versions
description: Use when updating version numbers in SharePoint Framework (SPFx) solutions after features or bugfixes
---

# Updating SPFx Versions

## Overview

SPFx solutions have version numbers in multiple files that must stay synchronized. This skill ensures all version locations are updated correctly.

## When to Use

- After implementing a new feature (bump minor version)
- After fixing a bug (bump patch version)
- Before deploying a new release
- When user asks to "increase version" or "bump version"

## Version Locations

| File | Property | Format | Notes |
|------|----------|--------|-------|
| `config/package-solution.json` | `solution.version` | `X.Y.Z.0` | 4-part, last is always 0 |
| `config/package-solution.json` | `solution.features[].version` | `X.Y.Z.0` | Must match solution version |
| `package.json` | `version` | `X.Y.Z` | 3-part semver |
| `*.manifest.json` | `version` | Usually `*` | Inherits from solution if `*` |

## Version Format

```
Major.Minor.Patch.Build
  │      │     │     └── Always 0 for SPFx
  │      │     └── Bugfixes
  │      └── New features
  └── Breaking changes
```

## Quick Reference

| Change Type | Version Bump | Example |
|-------------|--------------|---------|
| New feature | Minor | 2.20.0.0 → 2.21.0.0 |
| Bugfix | Patch | 2.21.0.0 → 2.21.1.0 |
| Breaking change | Major | 2.21.0.0 → 3.0.0.0 |

## Implementation

1. **Read current version** from `config/package-solution.json`
2. **Determine bump type** (major/minor/patch) based on change
3. **Update package-solution.json** - both `solution.version` AND `features[].version`
4. **Update package.json** - use 3-part format (drop the `.0`)
5. **Check manifest files** - only update if not using `*` wildcard

### Example Edit

```json
// config/package-solution.json - update BOTH locations
"version": "2.21.0.0"  →  "version": "2.21.1.0"

// package.json - 3-part format
"version": "2.21.0"  →  "version": "2.21.1"
```

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Only updating solution.version | Also update features[].version |
| Forgetting package.json | Always sync package.json |
| Using 3-part in package-solution.json | SPFx requires 4-part: X.Y.Z.0 |
| Using 4-part in package.json | npm requires 3-part: X.Y.Z |
| Updating manifest when it uses `*` | Leave `*` alone - it inherits |

## Verification

After updating, run `npm run build` or `gulp bundle` to verify no errors.
