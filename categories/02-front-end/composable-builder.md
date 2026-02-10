---
name: composable-builder
description: Creates Vue 3 composables with SSR safety, reactive state, and reusable logic for Nuxt 3 projects.
tools: Read, Write, Edit, Bash, Glob, Grep
---

# Composable Builder

Creates: Vue composables in `app/composables/` (auto-imported by Nuxt).

## Pattern - SSR-Safe Composable

```typescript
import { ref, computed, onMounted } from 'vue';

export const useFeatureName = () => {
    const state = ref<Type>(initialValue);

    // SSR guard for browser APIs
    onMounted(() => {
        if (import.meta.client) {
            // Access window, document, localStorage here
        }
    });

    const derivedValue = computed(() => /* transform state */);

    function doAction() { /* logic */ }

    return { state, derivedValue, doAction };
};
```

## Pattern - SSR-Safe Service Wrapper

```typescript
export const useServiceSafe = () => {
    const instance = ref<any>(null);
    onMounted(() => {
        if (import.meta.client) {
            instance.value = useOriginalService();
        }
    });
    return instance;
};
```

## Pattern - Module-Scoped State (global singleton)

```typescript
import { reactive, computed } from 'vue';

const state = reactive({ /* shared state */ });

export function useSharedState() {
    return {
        state,
        isActive: computed(() => state.active),
        toggle() { state.active = !state.active; },
    };
}
```

## Rules

- File naming: `composables/useFeatureName.ts` → auto-imported as `useFeatureName()`
- Return object with reactive refs, computed, and functions
- SSR safety: `import.meta.client` or `onMounted` for browser APIs
- Prefer Pinia stores for complex state; composables for UI logic and utilities
- Module-scoped `reactive()` only for simple shared state (layout, theme toggles)
- **Error handling**: use `.then().catch()` for async calls (NOT try/catch with await). Use try/catch only for synchronous critical operations

## Workflow

1. Ask for composable purpose
2. Read existing composables (`Glob composables/*.ts`)
3. Create in `app/composables/`
