# Fluent UI React v9 Setup

## Installation

```bash
npm install @fluentui/react-components
# or
yarn add @fluentui/react-components
```

## Basic Setup

### 1. Configure FluentProvider

Wrap your application root with `FluentProvider` and a theme:

```tsx
import { FluentProvider, webLightTheme } from '@fluentui/react-components';

function App() {
  return (
    <FluentProvider theme={webLightTheme}>
      <YourApp />
    </FluentProvider>
  );
}
```

### 2. Import Components

```tsx
import { Button, Input, Card } from '@fluentui/react-components';

function MyComponent() {
  return (
    <Card>
      <Input placeholder="Enter text" />
      <Button appearance="primary">Submit</Button>
    </Card>
  );
}
```

## Component Patterns

### Button Appearances

```tsx
<Button appearance="primary">Primary</Button>
<Button appearance="secondary">Secondary</Button>
<Button appearance="subtle">Subtle</Button>
<Button appearance="transparent">Transparent</Button>
<Button appearance="outline">Outline</Button>
```

### Form Inputs

```tsx
import { Input, Label, Field } from '@fluentui/react-components';

<Field label="Username" required>
  <Input placeholder="Enter username" />
</Field>
```

### Menu Pattern (Compound Components)

```tsx
import {
  Menu,
  MenuTrigger,
  MenuList,
  MenuItem,
  MenuPopover,
  Button,
} from '@fluentui/react-components';

<Menu>
  <MenuTrigger disableButtonEnhancement>
    <Button>Open Menu</Button>
  </MenuTrigger>
  <MenuPopover>
    <MenuList>
      <MenuItem>Option 1</MenuItem>
      <MenuItem>Option 2</MenuItem>
      <MenuItem>Option 3</MenuItem>
    </MenuList>
  </MenuPopover>
</Menu>
```

### Dialog Pattern

```tsx
import {
  Dialog,
  DialogTrigger,
  DialogSurface,
  DialogTitle,
  DialogBody,
  DialogActions,
  DialogContent,
  Button,
} from '@fluentui/react-components';

<Dialog>
  <DialogTrigger disableButtonEnhancement>
    <Button>Open Dialog</Button>
  </DialogTrigger>
  <DialogSurface>
    <DialogBody>
      <DialogTitle>Dialog Title</DialogTitle>
      <DialogContent>
        Dialog content goes here.
      </DialogContent>
      <DialogActions>
        <DialogTrigger disableButtonEnhancement>
          <Button appearance="secondary">Close</Button>
        </DialogTrigger>
        <Button appearance="primary">Confirm</Button>
      </DialogActions>
    </DialogBody>
  </DialogSurface>
</Dialog>
```

## Styling with Griffel

Fluent UI React v9 uses Griffel for CSS-in-JS styling:

```tsx
import { makeStyles, tokens } from '@fluentui/react-components';

const useStyles = makeStyles({
  container: {
    display: 'flex',
    gap: tokens.spacingHorizontalM,
    padding: tokens.spacingVerticalL,
    backgroundColor: tokens.colorNeutralBackground1,
  },
  highlight: {
    color: tokens.colorBrandForeground1,
    fontWeight: tokens.fontWeightSemibold,
  },
});

function MyComponent() {
  const styles = useStyles();
  
  return (
    <div className={styles.container}>
      <span className={styles.highlight}>Styled content</span>
    </div>
  );
}
```

## Common Tokens

### Spacing

- `tokens.spacingHorizontalXS` through `tokens.spacingHorizontalXXXL`
- `tokens.spacingVerticalXS` through `tokens.spacingVerticalXXXL`

### Colors

- `tokens.colorBrandBackground` - Primary brand color
- `tokens.colorNeutralBackground1` - Default background
- `tokens.colorNeutralForeground1` - Default text
- `tokens.colorPaletteRedBackground3` - Error states
- `tokens.colorPaletteGreenBackground3` - Success states

### Color Token Discovery

Palette color tokens don't follow a uniform naming pattern. When a token doesn't exist:

- Use TypeScript autocomplete to explore available tokens
- Try variations: `Border`, `BorderActive`, `Foreground`, `Background` + numbers (1, 2, 3)
- Example: `colorPaletteBlueBorder2` doesn't exist → try `colorPaletteBlueForeground2` or `colorPaletteBlueBorderActive`
- Available palettes: `Red`, `Green`, `Blue`, `Yellow`, `DarkOrange`, `Marigold`, `Teal`, `Purple`, etc.

### Typography

- `tokens.fontSizeBase100` through `tokens.fontSizeBase600`
- `tokens.fontWeightRegular`, `tokens.fontWeightSemibold`, `tokens.fontWeightBold`
- `tokens.lineHeightBase100` through `tokens.lineHeightBase600`

### Border Radius

- `tokens.borderRadiusSmall`
- `tokens.borderRadiusMedium`
- `tokens.borderRadiusLarge`
- `tokens.borderRadiusCircular`

## Dark Mode Support

```tsx
import { FluentProvider, webLightTheme, webDarkTheme } from '@fluentui/react-components';

function App() {
  const [isDark, setIsDark] = useState(false);
  
  return (
    <FluentProvider theme={isDark ? webDarkTheme : webLightTheme}>
      <YourApp />
    </FluentProvider>
  );
}
```

## Icons

Install the icons package:

```bash
npm install @fluentui/react-icons
```

Usage:

```tsx
import { AddRegular, DeleteRegular, EditRegular } from '@fluentui/react-icons';
import { Button } from '@fluentui/react-components';

<Button icon={<AddRegular />}>Add Item</Button>
<Button icon={<DeleteRegular />} appearance="subtle" />
```

## DataGrid

```tsx
import {
  DataGrid,
  DataGridHeader,
  DataGridHeaderCell,
  DataGridBody,
  DataGridRow,
  DataGridCell,
  TableColumnDefinition,
  createTableColumn,
} from '@fluentui/react-components';

type Item = { id: string; name: string; status: string };

const columns: TableColumnDefinition<Item>[] = [
  createTableColumn<Item>({
    columnId: 'name',
    renderHeaderCell: () => 'Name',
    renderCell: (item) => item.name,
  }),
  createTableColumn<Item>({
    columnId: 'status',
    renderHeaderCell: () => 'Status',
    renderCell: (item) => item.status,
  }),
];

<DataGrid items={items} columns={columns}>
  <DataGridHeader>
    <DataGridRow>
      {({ renderHeaderCell }) => (
        <DataGridHeaderCell>{renderHeaderCell()}</DataGridHeaderCell>
      )}
    </DataGridRow>
  </DataGridHeader>
  <DataGridBody<Item>>
    {({ item, rowId }) => (
      <DataGridRow<Item> key={rowId}>
        {({ renderCell }) => (
          <DataGridCell>{renderCell(item)}</DataGridCell>
        )}
      </DataGridRow>
    )}
  </DataGridBody>
</DataGrid>
```
