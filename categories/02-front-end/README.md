# Front-end Agents (`whyll-agents-front`)

Agentes especializados para Nuxt 3 com Vue 3, Pinia, PrimeVue e TypeScript.

**Stack:** Nuxt 3 | Vue 3 | Pinia | Axios | PrimeVue | Tailwind CSS | TypeScript | @nuxtjs/i18n

**Prefixo:** `@whyll-agents-front:<agente>`

---

## Convencoes

| Convencao | Padrao |
|-----------|--------|
| SFC Order | `<script setup>` -> `<template>` -> `<style scoped>` |
| PrimeVue | Auto-importado, NUNCA importar componentes. So utilities: `FilterMatchMode`, `useToast`, `useConfirm` |
| Error Handling | `.then().catch()` para async. try/catch so para ops sincronas criticas |
| SSR Safety | `import.meta.client`, `<ClientOnly>`, `typeof window !== 'undefined'` |
| i18n | `useI18n()` + `t()` para todo texto visivel |
| State | Pinia stores para estado complexo; composables para logica UI |
| Styling | Tailwind + `:deep()` para PrimeVue overrides |

---

## Agentes

### `nuxt-project-builder`

Cria um projeto Nuxt 3 do zero com toda a stack configurada: Pinia, Axios, PrimeVue, TypeScript, i18n e tooling recomendado. Scaffolda `nuxt.config.ts`, stores, composables, layouts, plugins e pagina inicial.

**Exemplos:**

```text
@whyll-agents-front:nuxt-project-builder Create a new Nuxt project called "my-dashboard" with npm

@whyll-agents-front:nuxt-project-builder Create a Nuxt 3 project called "ecommerce-app" using pnpm

@whyll-agents-front:nuxt-project-builder Create a new Nuxt project "admin-panel" with Portuguese as default locale
```

---

### `page-builder`

Cria paginas Nuxt 3 em `app/pages/` seguindo file-based routing. Usa `definePageMeta` (sempre primeiro no script), Composition API, PrimeVue auto-importado e SSR safety. Suporta layouts customizados e navegacao programatica com `navigateTo()`.

**Exemplos:**

```text
@whyll-agents-front:page-builder Create page /dashboard/analytics with charts and data table

@whyll-agents-front:page-builder Create page /products/[uuid] for product detail view

@whyll-agents-front:page-builder Create login page at /auth/login with layout: false (no sidebar)
```

---

### `component-builder`

Cria componentes Vue 3 SFC em `app/components/` organizados por feature/dominio. Usa Composition API, props e emits tipados com generics, PrimeVue auto-importado e Tailwind. Segue convencao PascalCase com sufixos descritivos (*Widget, *Panel, *Table, *Card, *Form).

**Exemplos:**

```text
@whyll-agents-front:component-builder Create ProductListTable in products domain with filters and pagination

@whyll-agents-front:component-builder Create UserProfileCard in dashboard domain with avatar and stats

@whyll-agents-front:component-builder Create OrderStatusWidget in orders domain with status badge and timeline
```

---

### `store-builder`

Cria Pinia stores em `app/stores/` com state tipado, getters, actions e SSR safety. Suporta persistencia opcional via localStorage (com guard `import.meta.client`). Actions usam `.then().catch().finally()` para chamadas async.

**Exemplos:**

```text
@whyll-agents-front:store-builder Create store for managing products with filters, pagination and CRUD actions

@whyll-agents-front:store-builder Create useNotificationStore with localStorage persistence for read/unread state

@whyll-agents-front:store-builder Create useCartStore with items, totals getter, and add/remove/clear actions
```

---

### `composable-builder`

Cria composables Vue 3 em `app/composables/` (auto-importados pelo Nuxt). Encapsula logica reutilizavel com refs reativas, computed e funcoes. Usa `import.meta.client` ou `onMounted` para APIs do browser. Ideal para logica UI e utilitarios.

**Exemplos:**

```text
@whyll-agents-front:composable-builder Create useDebounce composable for search input delay

@whyll-agents-front:composable-builder Create useClipboard composable to copy text to clipboard with SSR safety

@whyll-agents-front:composable-builder Create useDarkMode composable that syncs with localStorage and HTML class
```

---

### `api-service-builder`

Cria a camada de servicos API com 4 niveis: Axios client (`app/utils/axios.ts`), Server proxy Nitro (`server/api/proxy/[...path].ts`), chamada segura server-side (`server/utils/api.ts`) e classes de servico para dados locais. Nunca expoe `API_TOKEN` no client.

**Exemplos:**

```text
@whyll-agents-front:api-service-builder Create Axios client and proxy for external API integration

@whyll-agents-front:api-service-builder Create API service layer for /products endpoints with CRUD operations

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

Cria layouts Nuxt (`app/layouts/`) e componentes de layout (Topbar, Sidebar, Menu, MenuItem, Footer, Configurator) com PrimeVue e dark mode. Usa Pinia `useLayoutStore` para estado do layout. Menu suporta itens aninhados e icones PrimeIcons.

**Exemplos:**

```text
@whyll-agents-front:layout-builder Create admin layout with sidebar, topbar and footer

@whyll-agents-front:layout-builder Create AppMenu component with nested navigation items for dashboard, products and orders

@whyll-agents-front:layout-builder Add dark mode toggle to AppTopbar with View Transitions API fallback
```

---

### `plugin-builder`

Cria plugins Nuxt em `app/plugins/` com controle de execucao (universal, client-only, server-only) via sufixo do arquivo. Suporta `enforce` order (`pre`, default, `post`) e registro de plugins Vue. Helpers ficam acessiveis via `useNuxtApp().$helperName`.

**Exemplos:**

```text
@whyll-agents-front:plugin-builder Create client-only plugin for Google Analytics tracking

@whyll-agents-front:plugin-builder Create universal PrimeVue theme plugin with enforce: 'pre'

@whyll-agents-front:plugin-builder Create a plugin that provides a global $formatDate helper
```

---

## Qualidade

### `quality-checker`

Audita projetos Nuxt 3 contra as convencoes dos builders. Escaneia 9 areas: Pages, Components, Stores, Composables, API Layer, i18n, Layouts, Plugins e Config. Usa grep/glob para detectar violacoes e gera relatorio com lista de problemas por area. Verifica imports PrimeVue indevidos, SSR safety, tipagem, i18n, naming conventions e seguranca de tokens.

**Exemplos:**

```text
@whyll-agents-front:quality-checker Audit the entire Nuxt project for convention violations

@whyll-agents-front:quality-checker Scan all pages and components for hardcoded strings that should use i18n

@whyll-agents-front:quality-checker Check stores and composables for SSR safety issues and async error handling patterns
```
