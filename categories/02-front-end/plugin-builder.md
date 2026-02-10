---
name: plugin-builder
description: Creates Nuxt 3 plugins with enforce order, client/server execution control, and Vue plugin registration.
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

## Rules

- PrimeVue core: `primevue.ts` with `enforce: 'pre'` (universal)
- PrimeVue services: `primevue-services.client.ts` with `enforce: 'post'` (client-only)
- No `import.meta.client` needed inside `.client.ts` files
- Provided helpers accessible via `useNuxtApp().$helperName`

## Workflow

1. Determine execution scope (universal, client, server)
2. Read existing plugins (`Glob plugins/*.ts`)
3. Create plugin with correct suffix and enforce order
