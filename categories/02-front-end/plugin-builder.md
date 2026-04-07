---
name: plugin-builder
description: Creates Nuxt 4 plugins with enforce order, client/server execution control, and Vue plugin registration.
tools: Read, Write, Edit, Bash, Glob, Grep
---

# Plugin Builder

Creates: Nuxt plugins in `app/plugins/`.

## Execution Control

| Suffix | Runs on |
|---|---|
| `name.ts` | Server + Client (universal) |
| `name.client.ts` | Client only |
| `name.server.ts` | Server only |

## Pattern

```typescript
export default defineNuxtPlugin({
    name: 'plugin-name',
    enforce: 'pre',    // 'pre' | undefined | 'post'
    setup(nuxtApp) {
        nuxtApp.vueApp.use(SomePlugin, options);
        // Or provide helpers:
        // return { provide: { helperName: (arg: string) => { ... } } }
    },
});
```

## Enforce Order

```
1. enforce: 'pre'   → Framework setup (PrimeVue theme)
2. (default)        → Normal plugins
3. enforce: 'post'  → Services needing framework (Toast, Confirmation) — use .client.ts
```

## PrimeVue 4 Theme Plugin

PrimeVue 4 uses design tokens instead of CSS file imports. The `@primevue/nuxt-module` handles most configuration, but for custom preset switching:

```typescript
// app/plugins/primevue-theme.client.ts
import { usePreset, updatePreset } from '@primeuix/themes';
import Aura from '@primeuix/themes/aura';

export default defineNuxtPlugin({
    name: 'primevue-theme',
    enforce: 'post',
    setup() {
        // Switch entire preset at runtime:
        // usePreset(Aura);

        // Update specific tokens at runtime:
        // updatePreset({
        //     semantic: {
        //         primary: { 50: '{indigo.50}', /* ... */ },
        //     },
        // });
    },
});
```

## Rules

- PrimeVue core: configured via `@primevue/nuxt-module` in `nuxt.config.ts` (no separate plugin needed)
- PrimeVue services: `primevue-services.client.ts` with `enforce: 'post'` (client-only)
- No `import.meta.client` needed inside `.client.ts` files
- Provided helpers accessible via `useNuxtApp().$helperName`
- **PrimeVue 4**: No CSS imports (`primevue/resources/themes/*` no longer exists). Themes configured via presets in nuxt.config.ts
- **PrimeVue 4**: `$primevue.changeTheme()` removed — use `usePreset()` / `updatePreset()` from `@primeuix/themes`

## Workflow

1. Determine execution scope (universal, client, server)
2. Read existing plugins (`Glob plugins/*.ts`)
3. Create plugin with correct suffix and enforce order
