---
name: quality-checker
description: Audits Nuxt 3 code for WhylLima front-end patterns and reports violations. Checks pages, components, stores, composables, API layer, i18n, layouts, plugins, and config.
tools: Read, Write, Edit, Bash, Glob, Grep
---

# Quality Checker

Audits Nuxt 3 code and reports convention violations.

## Scan Commands

```bash
# Pages: missing definePageMeta
grep -rL "definePageMeta" app/pages/**/*.vue 2>/dev/null

# Pages/Components: direct PrimeVue imports (MUST be auto-imported)
grep -rn "from 'primevue/" app/components/ app/pages/ app/layouts/ 2>/dev/null
grep -rn "from \"primevue/" app/components/ app/pages/ app/layouts/ 2>/dev/null

# Pages/Components: try-catch with await (should use .then().catch())
grep -rn "try {" app/pages/ app/components/ app/stores/ app/composables/ 2>/dev/null | grep -v node_modules

# Stores: browser APIs without import.meta.client guard
grep -rn "localStorage\|sessionStorage\|window\.\|document\." app/stores/*.ts 2>/dev/null | grep -v "import.meta.client"

# Composables: files not prefixed with "use"
ls app/composables/*.ts 2>/dev/null | grep -v "/use"

# i18n: hardcoded visible text in templates (English words between tags)
grep -rn ">[A-Z][a-z]\{2,\}.*</" app/pages/ app/components/ 2>/dev/null | grep -v "t(" | grep -v "{{" | grep -v node_modules

# i18n: missing useI18n in pages/components with visible text
grep -rL "useI18n\|t(" app/pages/**/*.vue 2>/dev/null

# Plugins: missing enforce order
grep -rL "enforce" app/plugins/*.ts 2>/dev/null

# Config: API secrets in public runtimeConfig
grep -n "apiToken\|apiSecret\|apiKey" nuxt.config.ts 2>/dev/null | grep "public"

# Components: not in feature folders (flat in components/)
ls app/components/*.vue 2>/dev/null
```

## Detection & Checks by Area

### Pages (`app/pages/**/*.vue`)

**Bad signs:** definePageMeta not first statement, direct PrimeVue imports, try-catch with await, hardcoded strings without `t()`, browser APIs without SSR guard, missing layout declaration.

**Must have:**
- `definePageMeta` as first statement in `<script setup>`
- `useI18n()` + `t()` for all user-facing text
- SSR guards (`import.meta.client`, `<ClientOnly>`) for browser APIs
- `navigateTo()` for programmatic navigation

**Must NOT have:**
- Direct PrimeVue component imports (`from 'primevue/...'`)
- `try { await ... } catch` (use `.then().catch()`)
- Hardcoded strings in `<template>`

---

### Components (`app/components/**/*.vue`)

**Bad signs:** camelCase/kebab-case filename, flat in `components/` without feature folder, untyped props/emits, direct PrimeVue imports, wrong SFC order.

**Must have:**
- PascalCase filename with descriptive suffix (`*Widget.vue`, `*Panel.vue`, `*Table.vue`, `*Card.vue`, `*Form.vue`)
- Feature folder (`components/{domain}/`)
- Typed props: `defineProps<{ name: string }>()`
- Typed emits: `defineEmits<{ close: [] }>()`
- SFC order: `<script setup>` → `<template>` → `<style scoped>`
- Type imports: `import type { X } from '@/types/...'`

**Must NOT have:**
- Direct PrimeVue component imports
- camelCase or kebab-case filenames
- Untyped `defineProps()` / `defineEmits()`

---

### Stores (`app/stores/*.ts`)

**Bad signs:** store name doesn't match file, untyped state, browser APIs without `import.meta.client` guard, try-catch with await, no loading/error state for async ops.

**Must have:**
- Naming: `useEntityStore` in file `stores/entity.ts`
- Typed state: `state: (): EntityState => ({...})`
- `loading` and `error` state for async actions
- `.then().catch().finally()` for async operations
- `import.meta.client` for localStorage/window/document

**Must NOT have:**
- `async/await` with `try/catch` in actions
- Browser APIs without `import.meta.client` guard
- Untyped state

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

**Bad signs:** API token in client code, direct external API calls from client, missing proxy, missing 401 interceptor.

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

**Bad signs:** using composable for layout state, non-standard icon format, dark mode without SSR guard.

**Must have:**
- Pinia `useLayoutStore` for layout state
- Icons: `pi pi-fw pi-{name}` format
- SSR guard (`import.meta.client`) for dark mode toggle
- Menu with `to` for routes, nested `items` for submenus

**Must NOT have:**
- Composable for layout state (use Pinia store)
- Direct PrimeVue imports
- Dark mode toggle without SSR guard

---

### Plugins (`app/plugins/*.ts`)

**Bad signs:** browser APIs in universal plugin, wrong enforce order, unnecessary `import.meta.client` in `.client.ts`.

**Must have:**
- Correct suffix: `.ts` (universal), `.client.ts` (client), `.server.ts` (server)
- PrimeVue core: `enforce: 'pre'` (universal)
- PrimeVue services (Toast, Confirm): `enforce: 'post'` (client-only)

**Must NOT have:**
- Browser APIs in universal `.ts` plugins
- `import.meta.client` inside `.client.ts` files (already client-only)
- `enforce: 'pre'` for service plugins (Toast, Confirm)

---

### Config (`nuxt.config.ts`)

**Bad signs:** missing modules, non-strict TypeScript, secrets in public config, missing i18n config.

**Must have:**
- Modules: `['@pinia/nuxt', '@nuxtjs/i18n', '@primevue/nuxt-module']`
- `typescript: { strict: true }`
- `runtimeConfig`: `apiBaseUrl` and `apiToken` server-only
- i18n: locales configured, `strategy: 'no_prefix'`

**Must NOT have:**
- API tokens/secrets in `runtimeConfig.public`
- Missing required modules
- `typescript: { strict: false }` or missing

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

### ❌ Pages - 3 issues
@whyll-agents-front:page-builder Fix dashboard
File: app/pages/dashboard.vue
Issues: definePageMeta not first, hardcoded strings, try/catch await

@whyll-agents-front:page-builder Fix login
File: app/pages/auth/login.vue
Issues: hardcoded strings without t()

### ✅ Stores - OK

### ❌ Components - 2 issues
@whyll-agents-front:component-builder Fix ProductCard
File: app/components/productCard.vue
Issues: camelCase filename (rename to ProductCard.vue), direct PrimeVue import

### ✅ Composables - OK
### ✅ API Layer - OK

### ❌ i18n - 1 issue
@whyll-agents-front:i18n-manager Fix translations
Files: locales/es.json
Issues: missing keys present in pt.json (orders.status.pending)

### ✅ Layouts - OK
### ✅ Plugins - OK
### ✅ Config - OK

Total: 3 areas need fixes (6 issues)
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
| Page | definePageMeta first, useI18n + t(), SSR guards | PrimeVue imports, try/catch await, hardcoded strings |
| Component | PascalCase + suffix, typed props/emits, feature folder | PrimeVue imports, camelCase name, untyped props |
| Store | useXStore naming, typed state, .then().catch(), loading/error | try/catch await, browser APIs without guard |
| Composable | use* prefix, SSR guard, return typed object | complex state (use Pinia), browser APIs without guard |
| API Layer | Proxy pattern, token server-only, 401 interceptor | Token in client, direct external API calls |
| i18n | t() for all text, keys in sync, nested by domain | Hardcoded strings, flat keys, missing translations |
| Layout | Pinia useLayoutStore, pi icons, SSR guard dark mode | Composable for layout state, PrimeVue imports |
| Plugin | Correct suffix, correct enforce order | Browser APIs in universal plugin, import.meta.client in .client.ts |
| Config | All modules, strict TS, server-only secrets | Secrets in public config, missing modules |

## Workflow

1. Run scan commands above
2. Read flagged files to confirm issues
3. Group findings by area
4. Output report in format above
5. List total areas and issues
