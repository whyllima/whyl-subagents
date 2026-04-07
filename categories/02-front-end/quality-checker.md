---
name: quality-checker
description: Audits Nuxt 4 code for WhylLima front-end patterns and reports violations. Checks pages, components, stores, composables, API layer, i18n, layouts, plugins, config, and PrimeVue 4 compliance.
tools: Read, Write, Edit, Bash, Glob, Grep
---

# Quality Checker

Audits Nuxt 4 code and reports convention violations.

## Scan Commands

```bash
# Pages: missing definePageMeta
grep -rL "definePageMeta" app/pages/**/*.vue 2>/dev/null

# Pages/Components: direct PrimeVue imports (MUST be auto-imported)
grep -rn "from 'primevue/" app/components/ app/pages/ app/layouts/ 2>/dev/null
grep -rn "from \"primevue/" app/components/ app/pages/ app/layouts/ 2>/dev/null

# PrimeVue 4: old component names (renamed)
grep -rn "<Calendar\|<Dropdown\|<InputSwitch\|<OverlayPanel\|<Sidebar\b\|<Chips\b\|<TabView\|<AccordionTab" app/ 2>/dev/null

# PrimeVue 4: removed .p-fluid class (use fluid prop or <Fluid> wrapper)
grep -rn "p-fluid" app/ 2>/dev/null

# PrimeVue 4: deprecated inputStyle (use inputVariant)
grep -rn "inputStyle" nuxt.config.ts app/ 2>/dev/null

# PrimeVue 4: removed $primevue.changeTheme() (use usePreset/updatePreset)
grep -rn "changeTheme\|\\$primevue" app/ 2>/dev/null

# PrimeVue 4: old CSS imports (removed in v4)
grep -rn "primevue/resources\|primevue.min.css\|primevue.css" app/ nuxt.config.ts 2>/dev/null

# PrimeVue 4: old API imports (moved to @primevue/core/api)
grep -rn "from 'primevue/api'\|from \"primevue/api\"" app/ 2>/dev/null

# Pages/Components: try-catch with await (should use .then().catch())
grep -rn "try {" app/pages/ app/components/ app/stores/ app/composables/ 2>/dev/null | grep -v node_modules

# Stores: browser APIs without import.meta.client guard
grep -rn "localStorage\|sessionStorage\|window\.\|document\." app/stores/*.ts 2>/dev/null | grep -v "import.meta.client"

# Composables: files not prefixed with "use"
ls app/composables/*.ts 2>/dev/null | grep -v "/use"

# i18n: hardcoded visible text in templates (English words between tags)
grep -rn ">[A-Z][a-z]\{2,\}.*</" app/pages/ app/components/ 2>/dev/null | grep -v "t(" | grep -v "{{" | grep -v node_modules

# Plugins: missing enforce order
grep -rL "enforce" app/plugins/*.ts 2>/dev/null

# Config: API secrets in public runtimeConfig
grep -n "apiToken\|apiSecret\|apiKey" nuxt.config.ts 2>/dev/null | grep "public"

# Components: not in feature folders (flat in components/)
ls app/components/*.vue 2>/dev/null

# Nuxt 4: using null instead of undefined for defaults
grep -rn "error: null\|data: null" app/stores/ app/composables/ 2>/dev/null

# Nuxt 4: old Web API naming (statusCode/statusMessage)
grep -rn "statusCode\|statusMessage" server/ 2>/dev/null

# Nuxt 4: missing shared/ directory awareness
ls shared/ 2>/dev/null
```

## Detection & Checks by Area

### Pages (`app/pages/**/*.vue`)

**Bad signs:** definePageMeta not first statement, direct PrimeVue imports, try-catch with await, hardcoded strings without `t()`, browser APIs without SSR guard, missing layout declaration, old PrimeVue 3 component names.

**Must have:**
- `definePageMeta` as first statement in `<script setup>`
- `useI18n()` + `t()` for all user-facing text
- SSR guards (`import.meta.client`, `<ClientOnly>`) for browser APIs
- `navigateTo()` for programmatic navigation
- PrimeVue 4 component names (`<Select>` not `<Dropdown>`, `<DatePicker>` not `<Calendar>`, etc.)

**Must NOT have:**
- Direct PrimeVue component imports (`from 'primevue/...'`)
- `try { await ... } catch` (use `.then().catch()`)
- Hardcoded strings in `<template>`
- Old PrimeVue 3 component names

---

### Components (`app/components/**/*.vue`)

**Bad signs:** camelCase/kebab-case filename, flat in `components/` without feature folder, untyped props/emits, direct PrimeVue imports, wrong SFC order, old component names, `.p-fluid` class usage.

**Must have:**
- PascalCase filename with descriptive suffix (`*Widget.vue`, `*Panel.vue`, `*Table.vue`, `*Card.vue`, `*Form.vue`)
- Feature folder (`components/{domain}/`)
- Typed props: `defineProps<{ name: string }>()`
- Typed emits: `defineEmits<{ close: [] }>()`
- SFC order: `<script setup>` → `<template>` → `<style scoped>`
- Type imports: `import type { X } from '@/types/...'`
- PrimeVue 4 names: `<Select>`, `<DatePicker>`, `<ToggleSwitch>`, `<Drawer>`, `<Popover>`, `<Tabs>`
- `fluid` prop or `<Fluid>` wrapper (not `.p-fluid` class)

**Must NOT have:**
- Direct PrimeVue component imports
- camelCase or kebab-case filenames
- Untyped `defineProps()` / `defineEmits()`
- `.p-fluid` CSS class
- `from 'primevue/api'` (use `@primevue/core/api`)

---

### Stores (`app/stores/*.ts`)

**Bad signs:** store name doesn't match file, untyped state, browser APIs without `import.meta.client` guard, try-catch with await, no loading/error state for async ops, `error: null` instead of `undefined`.

**Must have:**
- Naming: `useEntityStore` in file `stores/entity.ts`
- Typed state: `state: (): EntityState => ({...})`
- `loading` and `error` state for async actions
- `.then().catch().finally()` for async operations
- `import.meta.client` for localStorage/window/document
- `error: undefined` (Nuxt 4 convention, not `null`)

**Must NOT have:**
- `async/await` with `try/catch` in actions
- Browser APIs without `import.meta.client` guard
- Untyped state
- `error: null` (use `undefined`)

---

### Composables (`app/composables/*.ts`)

**Bad signs:** filename not prefixed with `use`, browser APIs without SSR guard, complex state that should be in Pinia, no return type.

**Must have:**
- `use` prefix: `useFeatureName.ts`
- SSR guard: `import.meta.client` or `onMounted` for browser APIs
- Return object with refs, computed, and functions

**Must NOT have:**
- Filename without `use` prefix
- Complex shared state (use Pinia instead)
- Browser APIs without SSR guard

---

### API Layer (`app/utils/`, `server/api/proxy/`, `server/utils/`)

**Bad signs:** API token in client code, direct external API calls from client, missing proxy, missing 401 interceptor, old statusCode/statusMessage naming.

**Must have:**
- Axios client with `baseURL: '/api/proxy'`
- Server proxy at `server/api/proxy/[...path].ts`
- Secure call utility at `server/utils/api.ts`
- Token in `runtimeConfig.apiToken` (server-only)
- Response interceptor handling 401 (clear token + redirect)

**Must NOT have:**
- API token/secret in client-accessible code
- Direct external API calls from browser
- Hardcoded API URLs (use runtimeConfig)
- `statusCode`/`statusMessage` (Nuxt 4: use `status`/`statusText`)

---

### i18n (`locales/*.json`, `app/**/*.vue`)

**Bad signs:** hardcoded strings in templates, missing keys across locale files, flat key structure, missing `t()` calls.

**Must have:**
- `useI18n()` + `t()` for ALL user-facing text
- Keys nested by feature domain: `featureName.section.key`
- All locale files in sync (same keys in pt.json, en.json, es.json)

**Must NOT have:**
- Hardcoded strings in `<template>`
- Flat key structure
- Locale files with different key sets

---

### Layouts (`app/layouts/*.vue`, `app/components/layout/`)

**Bad signs:** using composable for layout state, non-standard icon format, dark mode without SSR guard, old `.app-dark` class, `<Sidebar>` instead of `<Drawer>`.

**Must have:**
- Pinia `useLayoutStore` for layout state
- Icons: `pi pi-fw pi-{name}` format
- SSR guard (`import.meta.client`) for dark mode toggle
- Menu with `to` for routes, nested `items` for submenus
- Dark mode via `.p-dark` class (PrimeVue 4 convention)
- `<Drawer>` for mobile sidebar (not `<Sidebar>`)

**Must NOT have:**
- Composable for layout state (use Pinia store)
- Direct PrimeVue imports
- Dark mode toggle without SSR guard
- `.app-dark` class (PrimeVue 4 uses `.p-dark`)
- `$primevue.changeTheme()` (use `usePreset()`/`updatePreset()`)

---

### Plugins (`app/plugins/*.ts`)

**Bad signs:** browser APIs in universal plugin, wrong enforce order, unnecessary `import.meta.client` in `.client.ts`, PrimeVue CSS imports, `changeTheme` usage.

**Must have:**
- Correct suffix: `.ts` (universal), `.client.ts` (client), `.server.ts` (server)
- PrimeVue core: configured in `nuxt.config.ts` via `@primevue/nuxt-module`
- PrimeVue services (Toast, Confirm): `enforce: 'post'` (client-only)

**Must NOT have:**
- Browser APIs in universal `.ts` plugins
- `import.meta.client` inside `.client.ts` files (already client-only)
- PrimeVue CSS imports (`primevue/resources/themes/*` removed in v4)
- `$primevue.changeTheme()` calls

---

### Config (`nuxt.config.ts`)

**Bad signs:** missing modules, non-strict TypeScript, secrets in public config, old PrimeVue config, old compatibilityDate.

**Must have:**
- Modules: `['@pinia/nuxt', '@nuxtjs/i18n', '@primevue/nuxt-module']`
- `typescript: { strict: true }`
- `runtimeConfig`: `apiBaseUrl` and `apiToken` server-only
- i18n: locales configured, `strategy: 'no_prefix'`
- PrimeVue 4: theme with preset object (`preset: Aura`), NOT string
- PrimeVue 4: `darkModeSelector` in theme options
- PrimeVue 4: `cssLayer` for Tailwind integration
- `compatibilityDate` recent (2025+)

**Must NOT have:**
- API tokens/secrets in `runtimeConfig.public`
- Missing required modules
- `typescript: { strict: false }` or missing
- `inputStyle` (deprecated, use `inputVariant`)
- `theme: 'aura'` as string (must be preset object)
- PrimeVue CSS in `css` array (except `primeicons/primeicons.css`)

---

### PrimeVue 4 Compliance (cross-cutting)

**Scan all `app/**/*.vue` for:**

| Pattern | Issue | Fix |
|---------|-------|-----|
| `<Calendar` | Renamed | Use `<DatePicker>` |
| `<Dropdown` | Renamed | Use `<Select>` |
| `<InputSwitch` | Renamed | Use `<ToggleSwitch>` |
| `<OverlayPanel` | Renamed | Use `<Popover>` |
| `<Sidebar` (component) | Renamed | Use `<Drawer>` |
| `<Chips` | Renamed | Use `<InputChips>` |
| `<TabView` | Deprecated | Use `<Tabs>` structure |
| `<AccordionTab` | Deprecated | Use `<AccordionPanel>` structure |
| `p-fluid` | Removed class | Use `fluid` prop or `<Fluid>` |
| `inputStyle` | Deprecated | Use `inputVariant` |
| `changeTheme` | Removed | Use `usePreset()`/`updatePreset()` |
| `from 'primevue/api'` | Moved | Use `from '@primevue/core/api'` |
| `primevue/resources` | Removed | Configure theme via preset |

---

## Fix Prompts

For each issue found, generate a fix prompt pointing to the correct builder agent.

### Page
```
@whyll-agents-front:page-builder Fix {PageName}
File: app/pages/{path}.vue
Issues: {list issues}
```

### Component
```
@whyll-agents-front:component-builder Fix {ComponentName}
File: app/components/{domain}/{ComponentName}.vue
Issues: {list issues}
```

### Store
```
@whyll-agents-front:store-builder Fix {StoreName}
File: app/stores/{name}.ts
Issues: {list issues}
```

### Composable
```
@whyll-agents-front:composable-builder Fix {ComposableName}
File: app/composables/{name}.ts
Issues: {list issues}
```

### API Layer
```
@whyll-agents-front:api-service-builder Fix API layer
Files: app/utils/axios.ts, server/api/proxy/[...path].ts, server/utils/api.ts
Issues: {list issues}
```

### i18n
```
@whyll-agents-front:i18n-manager Fix translations
Files: locales/pt.json, locales/en.json, locales/es.json
Issues: {list issues}
```

### Layout
```
@whyll-agents-front:layout-builder Fix {LayoutComponent}
File: app/layouts/{name}.vue OR app/components/layout/{Name}.vue
Issues: {list issues}
```

### Plugin
```
@whyll-agents-front:plugin-builder Fix {PluginName}
File: app/plugins/{name}.ts
Issues: {list issues}
```

### Config
```
@whyll-agents-front:nuxt-project-builder Fix nuxt.config.ts
File: nuxt.config.ts
Issues: {list issues}
```

## Output Format

```
## Quality Audit: {Feature/Project}

### Pages - 3 issues
@whyll-agents-front:page-builder Fix dashboard
File: app/pages/dashboard.vue
Issues: definePageMeta not first, old <Dropdown> (use <Select>), hardcoded strings

@whyll-agents-front:page-builder Fix login
File: app/pages/auth/login.vue
Issues: hardcoded strings without t()

### Stores - OK

### Components - 2 issues
@whyll-agents-front:component-builder Fix ProductCard
File: app/components/productCard.vue
Issues: camelCase filename, .p-fluid class (use fluid prop), old <Calendar> (use <DatePicker>)

### PrimeVue 4 - 4 issues
Files with deprecated component names: dashboard.vue, ProductForm.vue
Files with .p-fluid class: OrderList.vue
Files with old API import: utils/filters.ts (use @primevue/core/api)

### Composables - OK
### API Layer - OK

### i18n - 1 issue
@whyll-agents-front:i18n-manager Fix translations
Files: locales/es.json
Issues: missing keys present in pt.json (orders.status.pending)

### Layouts - OK
### Plugins - OK
### Config - OK

Total: 4 areas need fixes (10 issues)
```

## Agent Mapping

| Area | Fix Agent |
|------|-----------|
| Page | `@whyll-agents-front:page-builder` |
| Component | `@whyll-agents-front:component-builder` |
| Store | `@whyll-agents-front:store-builder` |
| Composable | `@whyll-agents-front:composable-builder` |
| API Layer | `@whyll-agents-front:api-service-builder` |
| i18n | `@whyll-agents-front:i18n-manager` |
| Layout | `@whyll-agents-front:layout-builder` |
| Plugin | `@whyll-agents-front:plugin-builder` |
| Config | `@whyll-agents-front:nuxt-project-builder` |

## Checklist

| Component | Must Have | Must NOT Have |
|-----------|-----------|---------------|
| Page | definePageMeta first, useI18n + t(), SSR guards, PrimeVue 4 names | PrimeVue imports, try/catch await, hardcoded strings, old component names |
| Component | PascalCase + suffix, typed props/emits, feature folder, PrimeVue 4 names | PrimeVue imports, camelCase name, untyped props, .p-fluid, old names |
| Store | useXStore naming, typed state, .then().catch(), loading/error, undefined defaults | try/catch await, browser APIs without guard, null defaults |
| Composable | use* prefix, SSR guard, return typed object | complex state (use Pinia), browser APIs without guard |
| API Layer | Proxy pattern, token server-only, 401 interceptor, status/statusText | Token in client, direct external API, statusCode/statusMessage |
| i18n | t() for all text, keys in sync, nested by domain | Hardcoded strings, flat keys, missing translations |
| Layout | Pinia useLayoutStore, pi icons, .p-dark dark mode, `<Drawer>` | Composable for layout state, .app-dark, `<Sidebar>`, changeTheme() |
| Plugin | Correct suffix, correct enforce order, no CSS imports | Browser APIs in universal, PrimeVue CSS imports, changeTheme() |
| Config | All modules, strict TS, server-only secrets, PrimeVue 4 preset object | Secrets in public, inputStyle, theme as string, old CSS |

## Workflow

1. Run scan commands above
2. Read flagged files to confirm issues
3. Group findings by area (including PrimeVue 4 compliance)
4. Output report in format above
5. List total areas and issues
