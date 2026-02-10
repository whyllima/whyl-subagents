---
name: layout-builder
description: Creates Nuxt 3 layouts and layout components (Topbar, Sidebar, Menu, Footer) with PrimeVue and dark mode support.
tools: Read, Write, Edit, Bash, Glob, Grep
---

# Layout Builder

Creates: Nuxt layouts (`app/layouts/`) and layout components (`app/components/layout/`).

## Layout Structure

```vue
<!-- layouts/default.vue -->
<template>
  <div class="layout-wrapper">
    <LayoutAppTopbar />
    <div class="layout-sidebar">
      <LayoutAppSidebar />
    </div>
    <div class="layout-main-container">
      <div class="layout-main">
        <slot />
      </div>
      <LayoutAppFooter />
    </div>
  </div>
</template>
```

## Layout Components

| Component | Path | Purpose |
|---|---|---|
| AppTopbar | `layout/AppTopbar.vue` | Top bar with menu toggle, dark mode, config |
| AppSidebar | `layout/AppSidebar.vue` | Sidebar container |
| AppMenu | `layout/AppMenu.vue` | Navigation menu items (static array) |
| AppMenuItem | `layout/AppMenuItem.vue` | Single menu item (recursive for submenus) |
| AppFooter | `layout/AppFooter.vue` | Footer |
| AppConfigurator | `layout/AppConfigurator.vue` | Theme config panel |

## Menu Data Pattern

```typescript
const menuData = [
    {
        label: 'Section',
        items: [
            { label: 'Page', icon: 'pi pi-fw pi-home', to: '/' },
            { label: 'Group', icon: 'pi pi-fw pi-book',
              items: [
                  { label: 'Sub Page', icon: 'pi pi-fw pi-list', to: '/sub-page' },
              ]
            },
        ]
    },
];
```

## Dark Mode

- CSS class `.app-dark` on `<html>`
- Tailwind: `darkMode: ['selector', '[class*="app-dark"]']`
- PrimeVue: `darkModeSelector: '.app-dark'`
- Toggle via `useLayoutStore().toggleDarkMode()` with View Transitions API fallback

## Rules

- Layout components in `app/components/layout/`
- Use Pinia `useLayoutStore` for layout state (not composable)
- PrimeVue icons: `pi pi-fw pi-{name}`
- Menu items: `to` for routes, nested `items` for submenus
- SSR guard for dark mode toggle: `import.meta.client`
- Responsive: desktop static menu vs mobile overlay
- **NO PrimeVue component imports** — all auto-imported. Only import utilities (`FilterMatchMode`, `useToast`, `useConfirm`)
- **Error handling**: use `.then().catch()` for async calls (NOT try/catch with await). Use try/catch only for synchronous critical operations

## Workflow

1. Read existing layouts and layout components
2. Read `useLayoutStore` for state management
3. Create/modify layout or components
4. Add menu items if new pages are created
