---
name: layout-builder
description: Creates Nuxt 4 layouts and layout components (Topbar, Sidebar, Menu, Footer) with PrimeVue 4 and dark mode support.
tools: Read, Write, Edit, Bash, Glob, Grep
---

# Layout Builder

Creates: Nuxt layouts (`app/layouts/`) and layout components (`app/components/layout/`).

## Layout Components

| Component | Purpose |
|---|---|
| `layout/AppTopbar.vue` | Top bar: menu toggle, dark mode, config |
| `layout/AppSidebar.vue` | Sidebar container (uses `<Drawer>` on mobile) |
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

## Dark Mode (PrimeVue 4)

PrimeVue 4 uses `.p-dark` class on `<html>` (configurable via `darkModeSelector` in nuxt.config.ts):

```typescript
// Toggle dark mode
function toggleDarkMode() {
    document.documentElement.classList.toggle('p-dark');
}
```

Tailwind config for PrimeVue 4 dark mode:

```js
// tailwind.config.js
export default {
    darkMode: ['selector', '[class*="p-dark"]'],
}
```

SSR guard: `import.meta.client` required for dark mode toggle.

## Layout Props (Nuxt 4)

Layouts can receive typed props:

```vue
<!-- app/layouts/admin.vue -->
<script setup lang="ts">
const props = defineProps<{
  showSidebar?: boolean
  breadcrumbs?: string[]
}>()
</script>
```

Pages pass props via `definePageMeta`:
```ts
definePageMeta({
  layout: 'admin',
  layoutProps: { showSidebar: true, breadcrumbs: ['Dashboard'] },
})
```

## Route Rule Layouts (Nuxt 4)

Set layouts declaratively via route rules in `nuxt.config.ts`:

```ts
routeRules: {
  '/admin/**': { appLayout: 'admin' },
  '/auth/**': { appLayout: 'blank' },
}
```

## PrimeVue 4 Component Notes

- `<Sidebar>` is now `<Drawer>` — use `<Drawer>` for mobile overlay sidebar
- Theme switching: use `usePreset()` / `updatePreset()` instead of `$primevue.changeTheme()`

## Rules

- Use Pinia `useLayoutStore` for layout state (not composable)
- Icons: `pi pi-fw pi-{name}`
- Menu: `to` for routes, nested `items` for submenus
- Responsive: desktop static menu vs mobile overlay (`<Drawer>`)
- **NO PrimeVue component imports** — all auto-imported. Only import utilities
- **Error handling**: `.then().catch()` for async calls

## Workflow

1. Read existing layouts and layout components
2. Read `useLayoutStore` for state management
3. Create/modify layout or components
4. Add menu items if new pages are created
