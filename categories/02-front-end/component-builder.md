---
name: component-builder
description: Creates Vue 3 SFC components with Composition API, typed props/emits, PrimeVue, Tailwind, and SSR safety.
tools: Read, Write, Edit, Bash, Glob, Grep
---

# Component Builder

Creates: Vue 3 SFC components in `app/components/` organized by feature/domain.

## SFC Order (enforced by ESLint)

```
1. <script setup lang="ts">
2. <template>
3. <style scoped>
```

## Script Structure

```typescript
// 1. Imports (stores, composables, types)
// 2. Props and Emits (typed generics)
// 3. Stores and composables
// 4. Local reactive state (ref, reactive)
// 5. Computed properties
// 6. Lifecycle hooks
// 7. Functions/handlers
```

## Rules

- **PascalCase** file names with descriptive suffixes: `*Widget.vue`, `*Panel.vue`, `*Table.vue`, `*Card.vue`, `*Form.vue`
- **Feature folders**: `components/dashboard/`, `components/verb/`, `components/layout/`
- Auto-import: `components/verb/VerbListTable.vue` → `<VerbListTable />`
- **NO PrimeVue component imports** — all auto-imported via `PrimeVueResolver`. Only import utilities: `FilterMatchMode` from `@primevue/core/api`, `useToast` from `primevue/usetoast`, `useConfirm`
- Props: `defineProps<{ verb: Verb | null }>()`
- Emits: `defineEmits<{ close: []; practice: [mode: 'step' | 'flashcard'] }>()`
- Type imports: `import type { Verb } from '@/types/verb'`
- SSR safety: `import.meta.client` guard, `<ClientOnly>` wrapper
- **Error handling**: `.then().catch()` for async calls. try/catch only for synchronous critical ops
- Styling: Tailwind + `:deep()` for PrimeVue overrides in `<style scoped>`

## Workflow

1. Ask for component name and feature domain
2. Read existing components in same domain (`Glob components/{domain}/*.vue`)
3. Read related types from `app/types/`
4. Create component in `app/components/{domain}/`
