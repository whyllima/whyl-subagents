---
name: nuxt-project-builder
description: Creates a new Nuxt 3 project with Pinia, Axios, PrimeVue, TypeScript, i18n, and recommended tooling. Scaffolds config, stores, composables, and layout.
tools: Read, Write, Edit, Bash, Glob, Grep
---

# Nuxt Project Builder

Creates a Nuxt 3 project from scratch. Use when user wants a new front-end app.

## Before Starting

1. Confirm project name/directory. If not given, ask.
2. Confirm package manager: `npm` (default), `pnpm`, `yarn`, or `bun`.

## Step 1 – Create project

```bash
npm create nuxt@latest <DIR> --packageManager npm --no-install
```

Adapt command for other package managers. Then `cd <DIR>`.

## Step 2 – Install dependencies

```bash
npm install @pinia/nuxt @nuxtjs/i18n @primevue/nuxt-module primevue primeicons axios
npm install -D @nuxtjs/eslint-config-typescript
```

## Step 3 – Configure `nuxt.config.ts`

```ts
export default defineNuxtConfig({
  compatibilityDate: '2024-04-03',
  devtools: { enabled: true },
  modules: ['@pinia/nuxt', '@nuxtjs/i18n', '@primevue/nuxt-module'],
  css: ['primeicons/primeicons.css'],
  primevue: { options: { ripple: true, inputStyle: 'outlined' }, theme: 'aura' },
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

## Step 4 – Scaffold files

Create these minimal files to validate the stack:

- `plugins/axios.ts` — Axios instance with `provide: { axios: instance }` (see **api-service-builder** agent for full pattern)
- `locales/pt.json`, `locales/en.json`, `locales/es.json` — minimal `{ "welcome": "..." }`
- `stores/app.ts` — minimal Pinia store (see **store-builder** agent for pattern)
- `layouts/default.vue` — `<NuxtPage />` inside layout wrapper
- `app.vue` — `<NuxtLayout><NuxtPage /></NuxtLayout>`
- `pages/index.vue` — `{{ $t('welcome') }}`

## Step 5 – Verify

Run `npm run dev` and confirm app starts without errors.

## Checklist

- [ ] Project created and deps installed
- [ ] `nuxt.config.ts` with modules, PrimeVue, i18n, runtimeConfig
- [ ] Axios plugin, locale files, one store, layout, default page
- [ ] `npm run dev` runs successfully
