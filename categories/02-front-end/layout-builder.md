---
name: layout-builder
description: Creates Nuxt 3 layouts and layout components (Topbar, Sidebar, Menu, Footer) with PrimeVue and dark mode support.
tools: Read, Write, Edit, Bash, Glob, Grep
---

# Layout Builder

Creates: Nuxt layouts (`app/layouts/`) and layout components (`app/components/layout/`).

## Layout Components

| Component | Purpose |
|---|---|
| `layout/AppTopbar.vue` | Top bar: menu toggle, dark mode, config |
| `layout/AppSidebar.vue` | Sidebar container |
| `layout/AppMenu.vue` | Navigation menu items (static array) |
| `layout/AppMenuItem.vue` | Single menu item (recursive for submenus) |
| `layout/AppFooter.vue` | Footer |
| `layout/AppConfigurator.vue` | Theme config panel |

## Menu Data Pattern

```typescript
const menuData = [
    { label: 'Section', items: [
        { label: 'Page', icon: 'pi pi-fw pi-home', to: '/' },
        { label: 'Group', icon: 'pi pi-fw pi-book', items: [
            { label: 'Sub', icon: 'pi pi-fw pi-list', to: '/sub' },
        ]},
    ]},
];
```

## Dark Mode

- Class `.app-dark` on `<html>` — Tailwind: `darkMode: ['selector', '[class*="app-dark"]']`
- Toggle via `useLayoutStore().toggleDarkMode()` with View Transitions API fallback
- SSR guard: `import.meta.client` required

## Rules

- Use Pinia `useLayoutStore` for layout state (not composable)
- Icons: `pi pi-fw pi-{name}`
- Menu: `to` for routes, nested `items` for submenus
- Responsive: desktop static menu vs mobile overlay
- **NO PrimeVue component imports** — all auto-imported. Only import utilities
- **Error handling**: `.then().catch()` for async calls

## Workflow

1. Read existing layouts and layout components
2. Read `useLayoutStore` for state management
3. Create/modify layout or components
4. Add menu items if new pages are created
