---
name: using-fluent-ui
description: Use when building React or Blazor applications that need Microsoft Fluent 2 design system components, enterprise UI consistency, or accessibility-first interfaces
---

# Using Fluent UI v2

## Overview

Build consistent, accessible, enterprise-grade interfaces using Microsoft's Fluent 2 Design System. This skill covers Fluent UI implementation for both React and Blazor applications.

**Core principle:** Consistency and accessibility through design tokens, not custom styling.

## When to Use

- Building applications that integrate with Microsoft ecosystem
- Enterprise applications requiring consistent, professional UI
- Applications where accessibility is a primary requirement
- Teams standardizing on Fluent design language
- Migrating from older Fluent UI versions to v2

## Core Concepts

### Design Tokens

Fluent UI uses design tokens for theming - CSS variables that control colors, spacing, typography, and more. Never hardcode values; always use tokens.

### Provider Pattern

Both React and Blazor require a provider component at the root that supplies theme context to all child components.

### Accessibility Built-In

Fluent components include ARIA attributes, keyboard navigation, and focus management by default. Don't override these unless necessary.

## Platform-Specific Setup

For detailed implementation guidance, see:
- **React**: [react-setup.md](react-setup.md)
- **Blazor**: [blazor-setup.md](blazor-setup.md)

## Theming

### Available Themes

| Theme | Use Case |
|-------|----------|
| `webLightTheme` | Standard light mode |
| `webDarkTheme` | Dark mode |
| `teamsLightTheme` | Teams integration (light) |
| `teamsDarkTheme` | Teams integration (dark) |

### Custom Theming

Create custom themes by extending base themes with your brand tokens:

```typescript
// React example
import { createLightTheme, BrandVariants } from '@fluentui/react-components';

const myBrand: BrandVariants = {
  10: '#020305',
  // ... define all 16 brand shades
  160: '#F0F4FA',
};

const myTheme = createLightTheme(myBrand);
```

## Component Patterns

### Layout Components

- **Card** - Container for related content
- **Divider** - Visual separator
- **Drawer** - Side panel overlay

### Input Components

- **Button** - Primary interaction element
- **Input** - Text input fields
- **Checkbox**, **Radio**, **Switch** - Selection controls
- **Dropdown**, **Combobox** - Selection from options
- **DatePicker**, **TimePicker** - Date/time selection

### Data Display

- **DataGrid** - Tabular data with sorting, filtering, virtualization
- **Table** - Simple tabular layout
- **Avatar** - User representation
- **Badge** - Status indicators

### Feedback

- **Toast** - Temporary notifications
- **Dialog** - Modal interactions
- **MessageBar** - Inline messages
- **Tooltip** - Contextual hints
- **Spinner**, **ProgressBar** - Loading states

## Best Practices

### DO

- ✅ Use the FluentProvider/providers at the app root
- ✅ Leverage design tokens for all styling
- ✅ Use compound components as designed (e.g., `Menu`, `MenuItem`, `MenuTrigger`)
- ✅ Test with keyboard navigation and screen readers
- ✅ Use appropriate `appearance` props (`primary`, `secondary`, `subtle`, `transparent`)

### DON'T

- ❌ Override component styles with custom CSS (breaks accessibility)
- ❌ Mix Fluent UI with conflicting design systems
- ❌ Ignore the component's built-in accessibility features
- ❌ Use deprecated v8/v0 patterns in v9 code
- ❌ Hardcode colors, spacing, or typography values

## Migration Notes

### From Fluent UI React v8 to v9

- Components are now tree-shakeable
- Styling moved from `styles` prop to `className` with Griffel
- Many components renamed (e.g., `DefaultButton` → `Button`)
- Theme structure completely redesigned

### From Bootstrap/Tailwind

- Remove utility class patterns
- Replace with Fluent component props and tokens
- Use `makeStyles` (React) or CSS variables (Blazor) for custom styling

## Anti-Patterns

❌ **Mixing design systems** - Don't combine Fluent with Bootstrap, Material, or Tailwind for the same components

❌ **Custom styling over tokens** - Avoid `style={{ color: '#0078d4' }}`, use tokens instead

❌ **Ignoring compound patterns** - Fluent components often work together (Menu + MenuTrigger + MenuList)

❌ **Skipping the provider** - Components won't theme correctly without FluentProvider/providers

## Resources

- [Fluent 2 Design System](https://fluent2.microsoft.design/)
- [Fluent UI React v9 Storybook](https://react.fluentui.dev/)
- [Fluent UI Blazor Documentation](https://www.fluentui-blazor.net/)
- [GitHub - Fluent UI React](https://github.com/microsoft/fluentui)
- [GitHub - Fluent UI Blazor](https://github.com/microsoft/fluentui-blazor)
