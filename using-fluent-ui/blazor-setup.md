# Fluent UI Blazor Setup

## Installation

### Using Templates (Recommended)

```bash
dotnet new install Microsoft.FluentUI.AspNetCore.Templates
dotnet new fluentblazor -n MyApp
```

### Manual Installation

```bash
dotnet add package Microsoft.FluentUI.AspNetCore.Components
```

## Basic Setup

### 1. Register Services

In `Program.cs`:

```csharp
builder.Services.AddFluentUIComponents();

// For Blazor Server, also add:
builder.Services.AddHttpClient();
```

### 2. Add Imports

In `_Imports.razor`:

```razor
@using Microsoft.FluentUI.AspNetCore.Components
```

### 3. Add Component Providers

At the end of `MainLayout.razor`:

```razor
<FluentToastProvider />
<FluentDialogProvider />
<FluentTooltipProvider />
<FluentMessageBarProvider />
<FluentMenuProvider />
```

### 4. Add Styles (Optional)

In `App.razor` or `_Layout.cshtml`:

```html
<link href="_content/Microsoft.FluentUI.AspNetCore.Components/css/reboot.css" rel="stylesheet" />
```

## Component Patterns

### Basic Components

```razor
<FluentCard Width="350px">
    <h2>Hello World!</h2>
    <FluentButton Appearance="Appearance.Accent">Click Me</FluentButton>
</FluentCard>
```

### Button Appearances

```razor
<FluentButton Appearance="Appearance.Accent">Accent</FluentButton>
<FluentButton Appearance="Appearance.Neutral">Neutral</FluentButton>
<FluentButton Appearance="Appearance.Outline">Outline</FluentButton>
<FluentButton Appearance="Appearance.Stealth">Stealth</FluentButton>
<FluentButton Appearance="Appearance.Lightweight">Lightweight</FluentButton>
```

### Form Inputs

```razor
<FluentTextField Label="Username" Required="true" @bind-Value="username" />
<FluentNumberField Label="Age" @bind-Value="age" />
<FluentCheckbox Label="Accept terms" @bind-Value="accepted" />
<FluentSwitch Label="Enable notifications" @bind-Value="notifications" />
```

### Select/Dropdown

```razor
<FluentSelect TOption="string" Label="Country" @bind-Value="selectedCountry">
    <FluentOption Value="us">United States</FluentOption>
    <FluentOption Value="uk">United Kingdom</FluentOption>
    <FluentOption Value="nl">Netherlands</FluentOption>
</FluentSelect>
```

### DataGrid

```razor
<FluentDataGrid Items="@people" Pagination="@pagination">
    <PropertyColumn Property="@(p => p.Name)" Sortable="true" />
    <PropertyColumn Property="@(p => p.Email)" />
    <PropertyColumn Property="@(p => p.Age)" Sortable="true" />
    <TemplateColumn Title="Actions">
        <FluentButton Appearance="Appearance.Stealth" 
                      OnClick="@(() => Edit(context))">
            Edit
        </FluentButton>
    </TemplateColumn>
</FluentDataGrid>

<FluentPaginator State="@pagination" />

@code {
    PaginationState pagination = new() { ItemsPerPage = 10 };
    IQueryable<Person> people = GetPeople().AsQueryable();
}
```

### DataGrid with EF Core

Install additional package:

```bash
dotnet add package Microsoft.FluentUI.AspNetCore.Components.DataGrid.EntityFrameworkAdapter
```

In `Program.cs`:

```csharp
builder.Services.AddDataGridEntityFrameworkAdapter();
```

### Dialog

```razor
@inject IDialogService DialogService

<FluentButton OnClick="@OpenDialog">Open Dialog</FluentButton>

@code {
    private async Task OpenDialog()
    {
        var dialog = await DialogService.ShowDialogAsync<MyDialogContent>(
            new DialogParameters
            {
                Title = "My Dialog",
                PrimaryAction = "Confirm",
                SecondaryAction = "Cancel",
                Width = "500px"
            });
        
        var result = await dialog.Result;
        if (!result.Cancelled)
        {
            // Handle confirmation
        }
    }
}
```

### Toast Notifications

```razor
@inject IToastService ToastService

<FluentButton OnClick="@ShowToast">Show Toast</FluentButton>

@code {
    private void ShowToast()
    {
        ToastService.ShowSuccess("Operation completed successfully!");
        ToastService.ShowError("An error occurred.");
        ToastService.ShowWarning("Please review your input.");
        ToastService.ShowInfo("New updates available.");
    }
}
```

### Message Bar

```razor
<FluentMessageBar Title="Information" Intent="MessageIntent.Info">
    This is an informational message.
</FluentMessageBar>

<FluentMessageBar Title="Success" Intent="MessageIntent.Success">
    Operation completed successfully.
</FluentMessageBar>

<FluentMessageBar Title="Warning" Intent="MessageIntent.Warning">
    Please review your input.
</FluentMessageBar>

<FluentMessageBar Title="Error" Intent="MessageIntent.Error">
    An error occurred.
</FluentMessageBar>
```

### Navigation Menu

```razor
<FluentNavMenu Width="250">
    <FluentNavLink Href="/" Icon="@(new Icons.Regular.Size20.Home())">
        Home
    </FluentNavLink>
    <FluentNavLink Href="/dashboard" Icon="@(new Icons.Regular.Size20.DataArea())">
        Dashboard
    </FluentNavLink>
    <FluentNavGroup Title="Settings" Icon="@(new Icons.Regular.Size20.Settings())">
        <FluentNavLink Href="/settings/profile">Profile</FluentNavLink>
        <FluentNavLink Href="/settings/security">Security</FluentNavLink>
    </FluentNavGroup>
</FluentNavMenu>
```

## Icons

Icons are included in the main package:

```razor
@using Microsoft.FluentUI.AspNetCore.Components.Icons

<FluentIcon Value="@(new Icons.Regular.Size20.Add())" />
<FluentIcon Value="@(new Icons.Filled.Size24.Delete())" Color="Color.Error" />

<FluentButton Icon="@(new Icons.Regular.Size20.Save())">
    Save
</FluentButton>
```

## Design Tokens

### Using Tokens in CSS

```css
.my-component {
    background-color: var(--neutral-layer-1);
    color: var(--neutral-foreground-1);
    padding: var(--spacing-vertical-m) var(--spacing-horizontal-m);
    border-radius: var(--control-corner-radius);
}
```

### Setting Tokens Programmatically

```razor
<FluentDesignSystemProvider AccentBaseColor="#0078d4"
                             NeutralBaseColor="#808080"
                             BaseLayerLuminance="1">
    <YourContent />
</FluentDesignSystemProvider>
```

## Render Modes

Fluent UI Blazor components require interactivity. For SSR pages, specify render mode:

```razor
@rendermode InteractiveServer
```

Or at component level:

```razor
<FluentButton @rendermode="InteractiveServer">Interactive Button</FluentButton>
```

## Common Patterns

### Loading State

```razor
<FluentProgressRing />

<FluentProgress />

<FluentSkeleton Width="200px" Height="20px" />
```

### Tabs

```razor
<FluentTabs>
    <FluentTab Label="Tab 1">
        Content for tab 1
    </FluentTab>
    <FluentTab Label="Tab 2">
        Content for tab 2
    </FluentTab>
    <FluentTab Label="Tab 3">
        Content for tab 3
    </FluentTab>
</FluentTabs>
```

### Accordion

```razor
<FluentAccordion>
    <FluentAccordionItem Heading="Section 1">
        Content for section 1
    </FluentAccordionItem>
    <FluentAccordionItem Heading="Section 2">
        Content for section 2
    </FluentAccordionItem>
</FluentAccordion>
```
