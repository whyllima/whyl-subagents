---
name: nuxt-project-builder
description: Creates a new Nuxt 4 project with Pinia, Axios, PrimeVue 4 (design tokens), TypeScript, i18n, and recommended tooling. Scaffolds config, stores, composables, and layout.
tools: Read, Write, Edit, Bash, Glob, Grep
---

# Nuxt Project Builder

Creates a Nuxt 4 project from scratch. Use when user wants a new front-end app.

## Before Starting

1. Confirm project name/directory. If not given, ask.
2. Confirm package manager: `npm` (default), `pnpm`, `yarn`, or `bun`.

## Step 1 – Create project

```bash
npx nuxi@latest init <DIR>
```

Adapt command for other package managers. Then `cd <DIR>`.

## Step 2 – Install dependencies

```bash
npm install @pinia/nuxt @nuxtjs/i18n @primevue/nuxt-module primevue @primeuix/themes primeicons axios
npm install -D @nuxtjs/eslint-config-typescript @nuxt/test-utils vitest
```

## Step 3 – Configure `nuxt.config.ts`

```ts
import Aura from '@primeuix/themes/aura'

export default defineNuxtConfig({
  compatibilityDate: '2025-01-01',
  devtools: { enabled: true },

  modules: ['@pinia/nuxt', '@nuxtjs/i18n', '@primevue/nuxt-module'],

  css: ['primeicons/primeicons.css'],

  primevue: {
    autoImport: true,
    options: {
      ripple: true,
      theme: {
        preset: Aura,
        options: {
          prefix: 'p',
          darkModeSelector: '.p-dark',
          cssLayer: {
            name: 'primevue',
            order: 'tailwind-base, primevue, tailwind-utilities',
          },
        },
      },
    },
  },

  i18n: {
    locales: [
      { code: 'pt', iso: 'pt-BR', file: 'pt.json', name: 'Português' },
      { code: 'en', iso: 'en-US', file: 'en.json', name: 'English' },
      { code: 'es', iso: 'es-ES', file: 'es.json', name: 'Español' },
    ],
    defaultLocale: 'pt',
    langDir: 'locales',
    strategy: 'no_prefix',
  },

  runtimeConfig: {
    apiBaseUrl: process.env.API_BASE_URL,
    apiToken: process.env.API_TOKEN,
    public: { appName: process.env.APP_NAME },
  },

  typescript: { strict: true },
})
```

## Step 4 – Directory structure (Nuxt 4)

Nuxt 4 uses `app/` as the default `srcDir`:

```
project-root/
  app/
    assets/
    components/{domain}/
    composables/
    layouts/
    middleware/
    pages/
    plugins/
    stores/
    types/
    utils/
    app.vue
    app.config.ts
  shared/
    utils/          ← auto-imported in app + server
    types/          ← auto-imported in app + server
  server/
    api/
    middleware/
    utils/
  locales/          ← i18n locale files
  public/
  nuxt.config.ts
```

Key aliases:
- `~` / `@` → `app/`
- `#shared` → `shared/`
- `#server` → `server/` (server context only)

## Step 5 – Scaffold files

Create these minimal files to validate the stack:

- `app/plugins/axios.ts` — Axios instance (see **api-service-builder** agent for full pattern)
- `locales/pt.json`, `locales/en.json`, `locales/es.json` — minimal `{ "welcome": "..." }`
- `app/stores/app.ts` — minimal Pinia store (see **store-builder** agent for pattern)
- `app/layouts/default.vue` — `<NuxtPage />` inside layout wrapper
- `app/app.vue` — `<NuxtLayout><NuxtPage /></NuxtLayout>`
- `app/pages/index.vue` — `{{ $t('welcome') }}`

## Step 6 – Verify

Run `npm run dev` and confirm app starts without errors.

## PrimeVue 4 Theme Customization

To customize the default preset:

```ts
import { definePreset } from '@primeuix/themes'
import Aura from '@primeuix/themes/aura'

const MyPreset = definePreset(Aura, {
  semantic: {
    primary: {
      50: '{indigo.50}',
      // ... 100 through 950
    },
    colorScheme: {
      light: { primary: { color: '{primary.500}' } },
      dark: { primary: { color: '{primary.400}' } },
    },
  },
})
```

Available presets: `Aura` (default), `Lara`, `Material`, `Nora`.

Dark mode toggle:
```ts
document.documentElement.classList.toggle('p-dark')
```

## Nuxt 4 Key Changes from Nuxt 3

- `useAsyncData`/`useFetch` return `shallowRef` data (use `deep: true` if needed)
- Default data/error is `undefined` (not `null`)
- Same-key data fetching is singleton (shared refs across components)
- `createUseFetch` / `createUseAsyncData` for custom composables with defaults
- Separate TypeScript projects per context (app, server, shared)
- `noUncheckedIndexedAccess: true` by default
- `shared/` directory auto-imported in both app and server
- `statusCode` → `status`, `statusMessage` → `statusText` in error handling

## Checklist

- [ ] Project created and deps installed (including `@primeuix/themes`)
- [ ] `nuxt.config.ts` with modules, PrimeVue 4 design tokens, i18n, runtimeConfig
- [ ] `app/` directory structure with `shared/` for cross-context code
- [ ] Axios plugin, locale files, one store, layout, default page
- [ ] `npm run dev` runs successfully
