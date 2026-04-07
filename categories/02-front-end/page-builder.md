---
name: page-builder
description: Creates Nuxt 4 pages with file-based routing, definePageMeta, layout assignment, SSR safety, and Composition API.
tools: Read, Write, Edit, Bash, Glob, Grep
---

# Page Builder

Creates: Nuxt 4 pages in `app/pages/` following file-based routing.

## Structure Order

```vue
<script setup lang="ts">
// 1. definePageMeta (MUST be first)
definePageMeta({ layout: 'default' });

// 2. Imports
// 3. Stores and composables
// 4. Reactive state (ref, reactive)
// 5. Computed properties
// 6. Lifecycle hooks (onMounted)
// 7. Functions/handlers
</script>

<template>
  <!-- PrimeVue 4 + Tailwind -->
</template>

<style scoped>
/* :deep() for PrimeVue overrides */
</style>
```

## Rules

- `definePageMeta` MUST be the first statement in `<script setup>`
- Layout `false` for pages without sidebar/topbar (e.g. login)
- Use `@/stores/` for Pinia stores, `#imports` for Nuxt auto-imports
- SSR guards: `import.meta.client` for browser APIs, `<ClientOnly>` in template
- Props/emits must be typed with generics: `defineProps<{}>()`, `defineEmits<{}>()`
- Use `navigateTo()` for programmatic navigation
- Use `useI18n()` + `t()` for all user-facing text
- File path = route: `pages/study/verb-trainer/all-verbs.vue` → `/study/verb-trainer/all-verbs`
- **NO PrimeVue component imports** — all auto-imported. Only import utilities (`FilterMatchMode` from `@primevue/core/api`, `useToast`, `useConfirm`)
- **Error handling**: use `.then().catch()` for async calls (NOT try/catch with await). Use try/catch only for synchronous critical operations

## Layout Props (Nuxt 4)

Layouts can receive typed props via `definePageMeta`:

```ts
definePageMeta({
  layout: 'admin',
  layoutProps: { showSidebar: true, breadcrumbs: ['Dashboard', 'Users'] },
})
```

## Route Groups (Nuxt 4)

Parenthesized folder names for convention-based authorization:

```
app/pages/(admin)/dashboard.vue  → /dashboard (meta.groups includes 'admin')
app/pages/(admin)/users.vue      → /users     (meta.groups includes 'admin')
app/pages/(public)/about.vue     → /about     (meta.groups includes 'public')
```

## Data Fetching (Nuxt 4)

- `useAsyncData` / `useFetch` return `shallowRef` data by default — use `deep: true` if you need deep reactivity
- Default data/error is `undefined` (not `null`)
- Same-key calls share refs (singleton behavior)
- Use `createUseFetch` for custom API composables with default options

```ts
// Data is shallowRef — trigger updates with .value reassignment
const { data: items, status } = await useFetch('/api/items')

// For deep reactivity when mutating nested props:
const { data: item } = await useFetch(`/api/items/${uuid}`, { deep: true })
```

## Workflow

1. Ask for route path if not provided
2. Read existing pages to match conventions (`Glob pages/**/*.vue`)
3. Create page file in correct `app/pages/` subdirectory
4. Verify page renders: `npx nuxi typecheck` or manual check
