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

## PrimeVue Auto-Import

**NEVER import PrimeVue components** — they are globally registered via `unplugin-vue-components` + `PrimeVueResolver`. Use directly in template: `<DataTable>`, `<Button>`, `<Card>`, `<Toast>`, `<Column>`, etc.

**Only these PrimeVue utilities need explicit import:**
- `import { FilterMatchMode } from '@primevue/core/api'`
- `import { useToast } from 'primevue/usetoast'`
- Other non-component utilities (e.g. `useConfirm`, `usePrimeVue`)

## Rules

- **PascalCase** file names: `VerbListTable.vue`, `StatsWidget.vue`
- **Suffixes**: `*Widget.vue`, `*Panel.vue`, `*Table.vue`, `*Card.vue`, `*Form.vue`
- **Feature folders**: `components/dashboard/`, `components/verb/`, `components/layout/`
- Auto-import resolution: `components/verb/VerbListTable.vue` → `<VerbVerbListTable />` or `<VerbListTable />`
- **NO PrimeVue component imports** — all auto-imported. Only import utilities (`FilterMatchMode`, `useToast`, `useConfirm`)
- Props: `defineProps<{ verb: Verb | null }>()`
- Emits: `defineEmits<{ close: []; practice: [mode: 'step' | 'flashcard'] }>()`
- Type imports: `import type { Verb } from '@/types/verb'`
- SSR safety: `import.meta.client` guard, `<ClientOnly>` wrapper, `typeof window !== 'undefined'`
- **Error handling**: use `.then().catch()` for async calls (NOT try/catch with await). Use try/catch only for synchronous critical operations
- Styling: Tailwind classes + `:deep()` for PrimeVue overrides in `<style scoped>`

## Workflow

1. Ask for component name and feature domain
2. Read existing components in same domain (`Glob components/{domain}/*.vue`)
3. Read related types from `app/types/`
4. Create component in `app/components/{domain}/`
