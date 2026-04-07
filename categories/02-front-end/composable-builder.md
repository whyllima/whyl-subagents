---
name: composable-builder
description: Creates Vue 3 composables with SSR safety, reactive state, and reusable logic for Nuxt 4 projects.
tools: Read, Write, Edit, Bash, Glob, Grep
---

# Composable Builder

Creates: Vue composables in `app/composables/` (auto-imported by Nuxt).

## Pattern

```typescript
import { ref, computed, onMounted } from 'vue';

export const useFeatureName = () => {
    const state = ref<Type>(initialValue);

    onMounted(() => {
        if (import.meta.client) {
            // Browser APIs here (window, document, localStorage)
            // Also use for SSR-safe service wrappers: instance.value = useToast()
        }
    });

    const derived = computed(() => /* transform state */);
    function doAction() { /* logic */ }

    return { state, derived, doAction };
};
```

For global singleton state, use module-scoped `reactive()`:

```typescript
const state = reactive({ active: false });
export function useSharedState() {
    return { state, isActive: computed(() => state.active) };
}
```

## Shared Composables (Nuxt 4)

For code shared between `app/` and `server/`:

```typescript
// shared/utils/format.ts — auto-imported in both app and server via #shared
export function formatCurrency(value: number, locale = 'pt-BR'): string {
    return new Intl.NumberFormat(locale, { style: 'currency', currency: 'BRL' }).format(value);
}
```

- `shared/utils/` and `shared/types/` are auto-imported in both app and server
- Cannot import Vue or Nitro-specific code in `shared/`
- Manual import via `#shared` alias: `import { formatCurrency } from '#shared/utils/format'`

## Custom Data Fetching Composable (Nuxt 4)

Use `createUseFetch` factory for custom API composables:

```typescript
// app/composables/useApi.ts
export const useApi = createUseFetch('/api/proxy', {
    headers: { Accept: 'application/json' },
    onRequest({ options }) {
        const token = useCookie('token');
        if (token.value) {
            options.headers.set('Authorization', `Bearer ${token.value}`);
        }
    },
    onResponseError({ response }) {
        if (response.status === 401) navigateTo('/auth/login');
    },
});
```

## Rules

- File naming: `composables/useFeatureName.ts` → auto-imported as `useFeatureName()`
- Return object with reactive refs, computed, and functions
- SSR safety: `import.meta.client` or `onMounted` for browser APIs
- Prefer Pinia stores for complex state; composables for UI logic and utilities
- Module-scoped `reactive()` only for simple shared state
- **Error handling**: `.then().catch()` for async calls. try/catch only for synchronous critical ops
- **Nuxt 4**: `useAsyncData`/`useFetch` return `shallowRef` — use `deep: true` if mutating nested properties
- **Nuxt 4**: Use `shared/utils/` for logic needed in both app and server

## Workflow

1. Ask for composable purpose
2. Read existing composables (`Glob composables/*.ts`)
3. Determine if it belongs in `app/composables/` or `shared/utils/`
4. Create in appropriate directory
