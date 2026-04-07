---
name: component-builder
description: Creates Vue 3 SFC components with Composition API, typed props/emits, PrimeVue 4, Tailwind, and SSR safety.
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
- **NO PrimeVue component imports** — all auto-imported via `@primevue/nuxt-module`. Only import utilities: `FilterMatchMode` from `@primevue/core/api`, `useToast` from `primevue/usetoast`, `useConfirm`
- Props: `defineProps<{ verb: Verb | null }>()`
- Emits: `defineEmits<{ close: []; practice: [mode: 'step' | 'flashcard'] }>()`
- Type imports: `import type { Verb } from '@/types/verb'`
- SSR safety: `import.meta.client` guard, `<ClientOnly>` wrapper
- **Error handling**: `.then().catch()` for async calls. try/catch only for synchronous critical ops
- Styling: Tailwind + `:deep()` for PrimeVue overrides in `<style scoped>`

## PrimeVue 4 Component Names

Use the new names (old names are deprecated):

| Old (PrimeVue 3) | New (PrimeVue 4) |
|---|---|
| `<Calendar>` | `<DatePicker>` |
| `<Dropdown>` | `<Select>` |
| `<InputSwitch>` | `<ToggleSwitch>` |
| `<OverlayPanel>` | `<Popover>` |
| `<Sidebar>` | `<Drawer>` |
| `<Chips>` | `<InputChips>` |
| `<TabView>` + `<TabPanel>` | `<Tabs>` + `<TabList>` + `<Tab>` + `<TabPanels>` + `<TabPanel>` |
| `<Accordion>` + `<AccordionTab>` | `<Accordion>` + `<AccordionPanel>` + `<AccordionHeader>` + `<AccordionContent>` |

## PrimeVue 4 Patterns

### Fluid inputs (replaces `.p-fluid` class)

```vue
<!-- Option A: fluid prop on individual components -->
<InputText fluid />
<Select fluid :options="items" />

<!-- Option B: Fluid wrapper -->
<Fluid>
  <InputText />
  <Select :options="items" />
</Fluid>
```

### Tabs (replaces TabView)

```vue
<Tabs value="0">
  <TabList>
    <Tab value="0">{{ t('tabs.general') }}</Tab>
    <Tab value="1">{{ t('tabs.settings') }}</Tab>
  </TabList>
  <TabPanels>
    <TabPanel value="0">General content</TabPanel>
    <TabPanel value="1">Settings content</TabPanel>
  </TabPanels>
</Tabs>
```

### Scoped Design Tokens (`dt` prop)

```vue
<Button :dt="{ root: { borderRadius: '2rem' } }" :label="t('action.save')" />
```

### Forms with @primevue/forms (optional)

```vue
<Form :resolver="zodResolver" @submit="onSubmit">
  <FormField name="email" v-slot="{ invalid, error }">
    <InputText name="email" :invalid="invalid" fluid />
    <Message v-if="error" severity="error" size="small">{{ error }}</Message>
  </FormField>
</Form>
```

## Workflow

1. Ask for component name and feature domain
2. Read existing components in same domain (`Glob components/{domain}/*.vue`)
3. Read related types from `app/types/`
4. Create component in `app/components/{domain}/`
