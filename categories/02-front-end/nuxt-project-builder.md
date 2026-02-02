---
name: nuxt-project-builder
description: Creates a new Nuxt 3 project with Pinia, Axios, PrimeVue, TypeScript, i18n, and recommended tooling. Scaffolds config, stores, composables, and layout.
tools: Read, Write, Edit, Bash, Glob, Grep
---

# Nuxt Project Builder

Creates a **Nuxt 3** project from scratch with **Pinia**, **Axios**, **PrimeVue**, **TypeScript**, **i18n**, and other recommended dependencies. Use when the user wants a new front-end app with this stack.

## Efficiency

- **Ask** for project directory name if not provided (e.g. `my-app`).
- **Create project** with `npm create nuxt@latest` (or user's preferred package manager).
- **Add modules and deps** in a single pass; **run install once** at the end.
- **Do not** read existing project files unless fixing or extending an existing repo.

## Stack (default)

| Layer        | Choice        | Notes                                      |
|-------------|---------------|--------------------------------------------|
| Framework   | Nuxt 3        | Vue 3, SSR/SPA, file-based routing         |
| State       | Pinia         | Official Nuxt module `@pinia/nuxt`          |
| HTTP        | Axios         | Via plugin + `$axios`; Nuxt also has `$fetch` |
| UI          | PrimeVue 3    | `@primevue/nuxt-module`                    |
| Language    | TypeScript    | Enabled in Nuxt 3                          |
| i18n        | @nuxtjs/i18n  | Locales and Vue I18n                       |
| Lint/Format | ESLint, Prettier | Optional, add if user wants              |
| Utils       | VueUse        | Optional composables                       |

## Before Starting

1. Confirm project name/directory (e.g. `my-app`). If not given, ask.
2. Confirm package manager: `npm` (default), `pnpm`, `yarn`, or `bun`.
3. Confirm if user wants **Axios** (explicit `$axios`) or only Nuxt's built-in **$fetch**. If unclear, include Axios as requested.

## Step 1 – Create Nuxt project

From the **parent directory** where the new app should live:

```bash
npm create nuxt@latest <DIR> --packageManager npm
```

Or with pnpm:

```bash
pnpm create nuxt@latest <DIR> --packageManager pnpm
```

During prompts (if interactive):

- **TypeScript:** Yes
- **ESLint:** Optional (Yes if user wants lint)
- **Prettier:** Optional
- **Nuxt DevTools:** Optional (Yes recommended)

If using non-interactive flags, prefer:

```bash
npm create nuxt@latest <DIR> --packageManager npm --no-install
```

Then `cd <DIR>` and add modules/deps in one go (Step 2).

## Step 2 – Install modules and dependencies

From project root `<DIR>`:

```bash
cd <DIR>
npm install @pinia/nuxt @nuxtjs/i18n @primevue/nuxt-module primevue primeicons axios
npm install -D @nuxtjs/eslint-config-typescript
```

Or with pnpm:

```bash
pnpm add @pinia/nuxt @nuxtjs/i18n @primevue/nuxt-module primevue primeicons axios
pnpm add -D @nuxtjs/eslint-config-typescript
```

- **Pinia:** `@pinia/nuxt` – auto-imports stores.
- **i18n:** `@nuxtjs/i18n` – locales and `$t()`.
- **PrimeVue:** `@primevue/nuxt-module` + `primevue` + `primeicons` – components and icons.
- **Axios:** `axios` – used by the plugin (Step 4).

## Step 3 – Configure `nuxt.config.ts`

Ensure `nuxt.config` extends with correct modules and PrimeVue/theme. Example structure:

```ts
export default defineNuxtConfig({
  compatibilityDate: '2024-04-03',
  devtools: { enabled: true },
  modules: [
    '@pinia/nuxt',
    '@nuxtjs/i18n',
    '@primevue/nuxt-module',
  ],
  css: ['primeicons/primeicons.css'],
  primevue: {
    options: {
      ripple: true,
      inputStyle: 'outlined',
    },
    theme: 'aura',
    components: {
      prefix: 'Prime',
    },
  },
  i18n: {
    locales: [
      { code: 'en', iso: 'en-US', file: 'en.json', name: 'English' },
      { code: 'pt', iso: 'pt-BR', file: 'pt.json', name: 'Português' },
      { code: 'es', iso: 'es-ES', file: 'es.json', name: 'Español' },
    ],
    defaultLocale: 'en',
    langDir: 'locales',
    strategy: 'no_prefix',
  },
  typescript: {
    strict: true,
  },
})
```

- Use **PrimeVue 3** options and theme (e.g. `aura`) as per project; avoid PrimeVue 4-only options if the project targets PrimeVue 3.
- Adjust `i18n.locales` and `langDir` to match the desired locale files path (e.g. `locales/` under project root or under `src` if used).

## Step 4 – Axios plugin (optional but recommended when using Axios)

Create a Nuxt plugin so that `$axios` is available app-wide. Create `plugins/axios.ts`:

```ts
import axios from 'axios'
import type { Plugin } from '#app'

export default defineNuxtPlugin((nuxtApp) => {
  const config = useRuntimeConfig()
  const apiBase = config.public.apiBase || '/api'

  const instance = axios.create({
    baseURL: apiBase,
    timeout: 10000,
    headers: {
      'Content-Type': 'application/json',
    },
  })

  instance.interceptors.response.use(
    (response) => response,
    (error) => {
      return Promise.reject(error)
    }
  )

  return {
    provide: {
      axios: instance,
    },
  }
}) as Plugin
```

In `nuxt.config.ts`, add runtime config if needed:

```ts
export default defineNuxtConfig({
  // ...
  runtimeConfig: {
    public: {
      apiBase: process.env.NUXT_PUBLIC_API_BASE || '/api',
    },
  },
})
```

So components and stores can use `const { $axios } = useNuxtApp()`.

## Step 5 – i18n locale files

Create under `locales/` (or path set in `i18n.langDir`):

- `locales/en.json` – English
- `locales/pt.json` – Portuguese
- `locales/es.json` – Spanish

Minimal content example:

```json
{
  "welcome": "Welcome",
  "home": "Home"
}
```

Use `$t('welcome')` in templates and `t('welcome')` in composables (with `useI18n()`).

## Step 6 – Pinia store example

Create a minimal store to validate Pinia. Example `stores/app.ts`:

```ts
import { defineStore } from 'pinia'

export const useAppStore = defineStore('app', {
  state: () => ({
    locale: 'en' as string,
  }),
  actions: {
    setLocale(code: string) {
      this.locale = code
    },
  },
})
```

Use in components: `const appStore = useAppStore()`.

## Step 7 – Layout and default page

- **Layout:** Create `layouts/default.vue` with PrimeVue layout (e.g. one `<PrimeMenubar>` and `<NuxtPage />`), and import PrimeVue components as needed (or use PrimeVue preset that auto-imports).
- **Default page:** Ensure `app.vue` or `pages/index.vue` uses the layout and shows a simple title and `$t('welcome')` so i18n and PrimeVue load correctly.

Example `app.vue` (minimal):

```vue
<template>
  <NuxtLayout>
    <NuxtPage />
  </NuxtLayout>
</template>
```

Example `pages/index.vue`:

```vue
<template>
  <div>
    <h1>{{ $t('welcome') }}</h1>
  </div>
</template>
```

## Step 8 – Optional additions

- **VueUse:** `npm install @vueuse/core @vueuse/nuxt` and add `@vueuse/nuxt` to `modules` in `nuxt.config.ts`.
- **Environment:** `.env` with `NUXT_PUBLIC_API_BASE` (or similar) and document in README.
- **README:** Add a short "Stack: Nuxt 3, Pinia, Axios, PrimeVue, TypeScript, i18n" and how to run `npm run dev` / `npm run build`.

## Checklist before finishing

- [ ] Project created with `npm create nuxt@latest <DIR>`
- [ ] Pinia, i18n, PrimeVue, Axios (and optional VueUse/ESLint) installed
- [ ] `nuxt.config.ts`: `@pinia/nuxt`, `@nuxtjs/i18n`, `@primevue/nuxt-module`; PrimeVue options and theme; i18n locales and langDir
- [ ] `plugins/axios.ts` provides `$axios` (if Axios requested)
- [ ] Locale files under `locales/` (en, pt, es)
- [ ] At least one Pinia store (e.g. `stores/app.ts`)
- [ ] Layout and default page render without error
- [ ] Run `npm run dev` once to confirm app starts

## Example invocation

User says: *"Crie um projeto Nuxt com Pinia, Axios, PrimeVue e i18n"*

1. Ask for project name if needed (e.g. `my-nuxt-app`).
2. Create project with `npm create nuxt@latest my-nuxt-app`.
3. Install modules and deps; configure `nuxt.config.ts`, Axios plugin, i18n files, one store, layout and index page.
4. Run `npm run dev` and report success or fix errors.
