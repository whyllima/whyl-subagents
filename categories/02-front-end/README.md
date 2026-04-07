# Front-end Agents (`whyll-agents-front`)

Agentes especializados para Nuxt 4 com Vue 3, Pinia, PrimeVue 4 e TypeScript.

**Stack:** Nuxt 4 | Vue 3 | Pinia | Axios | PrimeVue 4 (Design Tokens) | Tailwind CSS | TypeScript | @nuxtjs/i18n

**Prefixo:** `@whyll-agents-front:<agente>`

---

## Convencoes

| Convencao | Padrao |
|-----------|--------|
| SFC Order | `<script setup>` -> `<template>` -> `<style scoped>` |
| PrimeVue | Auto-importado via `@primevue/nuxt-module`, NUNCA importar componentes. So utilities: `FilterMatchMode` de `@primevue/core/api`, `useToast`, `useConfirm` |
| PrimeVue 4 Names | `<Select>` (nao Dropdown), `<DatePicker>` (nao Calendar), `<ToggleSwitch>` (nao InputSwitch), `<Drawer>` (nao Sidebar), `<Popover>` (nao OverlayPanel), `<Tabs>` (nao TabView) |
| PrimeVue 4 Theme | Design Tokens via presets (`Aura`, `Lara`, `Material`, `Nora`). Sem CSS imports. Dark mode via `.p-dark` |
| PrimeVue 4 Fluid | `fluid` prop ou `<Fluid>` wrapper (nao `.p-fluid` class) |
| Error Handling | `.then().catch()` para async. try/catch so para ops sincronas criticas |
| SSR Safety | `import.meta.client`, `<ClientOnly>`, `typeof window !== 'undefined'` |
| i18n | `useI18n()` + `t()` para todo texto visivel |
| State | Pinia stores para estado complexo; composables para logica UI |
| Styling | Tailwind + `:deep()` para PrimeVue overrides |
| Nuxt 4 Dir | `app/` como srcDir, `shared/` para codigo cross-context, `server/` na raiz |
| Data Fetching | `shallowRef` por padrao, `undefined` (nao `null`), singleton por key |

---

## Agentes

### `nuxt-project-builder`

Cria um projeto Nuxt 4 do zero com toda a stack configurada: Pinia, Axios, PrimeVue 4 (design tokens), TypeScript, i18n e tooling recomendado. Scaffolda `nuxt.config.ts` com preset, stores, composables, layouts, plugins e pagina inicial. Estrutura `app/` + `shared/` + `server/`.

**Exemplos:**

```text
@whyll-agents-front:nuxt-project-builder Create a new Nuxt project called "my-dashboard" with npm

@whyll-agents-front:nuxt-project-builder Create a Nuxt 4 project called "ecommerce-app" using pnpm

@whyll-agents-front:nuxt-project-builder Create a new Nuxt project "admin-panel" with Portuguese as default locale
```

---

### `page-builder`

Cria paginas Nuxt 4 em `app/pages/` seguindo file-based routing. Usa `definePageMeta` (sempre primeiro no script), Composition API, PrimeVue 4 auto-importado e SSR safety. Suporta layout props tipados, route groups e `navigateTo()`.

**Exemplos:**

```text
@whyll-agents-front:page-builder Create page /dashboard/analytics with charts and data table

@whyll-agents-front:page-builder Create page /products/[uuid] for product detail view

@whyll-agents-front:page-builder Create login page at /auth/login with layout: false (no sidebar)
```

---

### `component-builder`

Cria componentes Vue 3 SFC em `app/components/` organizados por feature/dominio. Usa Composition API, props e emits tipados com generics, PrimeVue 4 auto-importado (nomes novos: `<Select>`, `<DatePicker>`, `<Tabs>`, etc.) e Tailwind. Suporta `fluid` prop, `dt` prop para design tokens scoped e `@primevue/forms`.

**Exemplos:**

```text
@whyll-agents-front:component-builder Create ProductListTable in products domain with Select filters and pagination

@whyll-agents-front:component-builder Create UserProfileCard in dashboard domain with avatar and stats

@whyll-agents-front:component-builder Create OrderForm in orders domain with @primevue/forms and Zod validation
```

---

### `store-builder`

Cria Pinia stores em `app/stores/` com state tipado, getters, actions e SSR safety. Suporta persistencia opcional via localStorage (com guard `import.meta.client`). Actions usam `.then().catch().finally()`. Estado de erro usa `undefined` (Nuxt 4 convention).

**Exemplos:**

```text
@whyll-agents-front:store-builder Create store for managing products with filters, pagination and CRUD actions

@whyll-agents-front:store-builder Create useNotificationStore with localStorage persistence for read/unread state

@whyll-agents-front:store-builder Create useCartStore with items, totals getter, and add/remove/clear actions
```

---

### `composable-builder`

Cria composables Vue 3 em `app/composables/` (auto-importados pelo Nuxt). Encapsula logica reutilizavel com refs reativas, computed e funcoes. Suporta `shared/utils/` para codigo compartilhado app↔server e `createUseFetch` para API composables customizados (Nuxt 4).

**Exemplos:**

```text
@whyll-agents-front:composable-builder Create useDebounce composable for search input delay

@whyll-agents-front:composable-builder Create useApi composable using createUseFetch with auth interceptor

@whyll-agents-front:composable-builder Create shared/utils/formatCurrency for use in both app and server
```

---

### `api-service-builder`

Cria a camada de servicos API com 4 niveis: Axios client (`app/utils/axios.ts`), Server proxy Nitro (`server/api/proxy/[...path].ts`), chamada segura server-side (`server/utils/api.ts`) e classes de servico. Alternativa Nuxt 4: `createUseFetch` para data fetching SSR-compatible. Usa `#server` alias e `status`/`statusText` (Web API naming).

**Exemplos:**

```text
@whyll-agents-front:api-service-builder Create Axios client and proxy for external API integration

@whyll-agents-front:api-service-builder Create API composable using createUseFetch with Bearer auth

@whyll-agents-front:api-service-builder Create server proxy with Bearer token and 401 redirect handling
```

---

### `i18n-manager`

Gerencia arquivos de locale (`locales/`), chaves de traducao e configuracao do `@nuxtjs/i18n`. Organiza chaves por feature domain (objetos aninhados). Garante sincronia entre todos os idiomas (pt, en, es). Detecta strings hardcoded que deveriam usar `t()`.

**Exemplos:**

```text
@whyll-agents-front:i18n-manager Add translations for the "orders" feature in pt, en, and es

@whyll-agents-front:i18n-manager Sync all locale files and find missing keys across languages

@whyll-agents-front:i18n-manager Find hardcoded strings in components that should use i18n
```

---

### `layout-builder`

Cria layouts Nuxt (`app/layouts/`) e componentes de layout (Topbar, Sidebar, Menu, MenuItem, Footer, Configurator) com PrimeVue 4 e dark mode via `.p-dark`. Usa Pinia `useLayoutStore` para estado. `<Drawer>` para sidebar mobile. Suporta layout props tipados e route rule layouts (Nuxt 4).

**Exemplos:**

```text
@whyll-agents-front:layout-builder Create admin layout with Drawer sidebar, topbar and footer

@whyll-agents-front:layout-builder Create AppMenu component with nested navigation items

@whyll-agents-front:layout-builder Add dark mode toggle with .p-dark class and SSR safety
```

---

### `plugin-builder`

Cria plugins Nuxt em `app/plugins/` com controle de execucao (universal, client-only, server-only) via sufixo do arquivo. PrimeVue 4 configurado via `@primevue/nuxt-module` (sem plugin separado). Suporta `usePreset()`/`updatePreset()` de `@primeuix/themes` para troca de tema runtime.

**Exemplos:**

```text
@whyll-agents-front:plugin-builder Create client-only plugin for Google Analytics tracking

@whyll-agents-front:plugin-builder Create theme switcher plugin using usePreset from @primeuix/themes

@whyll-agents-front:plugin-builder Create a plugin that provides a global $formatDate helper
```

---

### `middleware-builder`

Cria route middleware Nuxt 4 em `app/middleware/` para auth guards, redirects e controle de acesso. Suporta route groups (Nuxt 4) para autorizacao convention-based. SSR-safe com `useCookie()` e `navigateTo()`.

**Exemplos:**

```text
@whyll-agents-front:middleware-builder Create auth guard that redirects to /auth/login

@whyll-agents-front:middleware-builder Create admin middleware using Nuxt 4 route groups

@whyll-agents-front:middleware-builder Create global token refresh middleware
```

---

### `testing-builder`

Cria testes Vitest para apps Nuxt 4: testes de componente com `mountSuspended`, testes de store com Pinia, testes de composable e testes de servico API com Axios mockado. Usa `#components` alias e `undefined` defaults (Nuxt 4).

**Exemplos:**

```text
@whyll-agents-front:testing-builder Create component test for ProductListTable

@whyll-agents-front:testing-builder Create store test for useCartStore with mock API calls

@whyll-agents-front:testing-builder Create composable test for useDebounce
```

---

## Qualidade

### `quality-checker`

Audita projetos Nuxt 4 contra as convencoes dos builders. Escaneia 10 areas: Pages, Components, Stores, Composables, API Layer, i18n, Layouts, Plugins, Config e **PrimeVue 4 Compliance**. Detecta nomes de componentes antigos (Dropdown→Select, Calendar→DatePicker), `.p-fluid` class, CSS imports removidos, `inputStyle` deprecated, `changeTheme()` removido, `from 'primevue/api'` (movido para `@primevue/core/api`), defaults `null` (Nuxt 4 usa `undefined`) e `statusCode`/`statusMessage` (usar `status`/`statusText`).

**Exemplos:**

```text
@whyll-agents-front:quality-checker Audit the entire Nuxt project for Nuxt 4 and PrimeVue 4 compliance

@whyll-agents-front:quality-checker Scan for deprecated PrimeVue 3 component names and patterns

@whyll-agents-front:quality-checker Check all stores and composables for Nuxt 4 convention compliance
```
