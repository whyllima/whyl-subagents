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
| `plugin.ts` | Server + Client (universal) |
| `plugin.client.ts` | Client only |
| `plugin.server.ts` | Server only |

## Pattern - Universal Plugin

```typescript
export default defineNuxtPlugin({
    name: 'plugin-name',
    enforce: 'pre',    // 'pre' runs before others, 'post' runs after
    setup(nuxtApp) {
        nuxtApp.vueApp.use(SomePlugin, options);
    },
});
```

## Pattern - Client-Only Plugin

```typescript
// plugins/services.client.ts
export default defineNuxtPlugin({
    name: 'services',
    enforce: 'post',
    setup(nuxtApp) {
        nuxtApp.vueApp.use(ToastService);
        nuxtApp.vueApp.use(ConfirmationService);
    },
});
```

## Pattern - Provide Helper

```typescript
export default defineNuxtPlugin((nuxtApp) => {
    return {
        provide: {
            helperName: (arg: string) => { /* logic */ },
        },
    };
});
```

## Enforce Order

```
1. enforce: 'pre'   → Framework setup (PrimeVue theme, core plugins)
2. (default)        → Normal plugins
3. enforce: 'post'  → Services that depend on framework (Toast, Confirmation)
```

## Rules

- PrimeVue core: `primevue.ts` with `enforce: 'pre'` (universal)
- PrimeVue services: `primevue-services.client.ts` with `enforce: 'post'` (client-only)
- Client-only plugins: suffix `.client.ts` — for browser-only services
- Provided helpers accessible via `useNuxtApp().$helperName`
- No `import.meta.client` needed inside `.client.ts` files (already client-only)

## Workflow

1. Determine execution scope (universal, client, server)
2. Read existing plugins (`Glob plugins/*.ts`)
3. Create plugin with correct suffix and enforce order
