---
name: store-builder
description: Creates Pinia stores with typed state, getters, actions, SSR safety, and optional localStorage persistence.
tools: Read, Write, Edit, Bash, Glob, Grep
---

# Store Builder

Creates: Pinia stores in `app/stores/` (auto-imported by `@pinia/nuxt`).

## Pattern

```typescript
import { defineStore } from 'pinia';
import type { EntityState } from '@/types/entity';

export const useEntityStore = defineStore('entity', {
    state: (): EntityState => ({
        items: [],
        loading: false,
        error: null as string | null,
        // filters, pagination, selection...
    }),

    getters: {
        filteredItems: (state) => { /* filter/sort logic */ },
        selectedItem: (state) => { /* find by ID */ },
    },

    actions: {
        loadItems() {
            this.loading = true;
            api.get('/items')
                .then((res) => {
                    this.items = res.data;
                })
                .catch((e: any) => {
                    this.error = e.message;
                })
                .finally(() => {
                    this.loading = false;
                });
        },
        // SSR-safe persistence
        loadFromStorage() {
            if (import.meta.client) {
                const stored = localStorage.getItem('key');
                if (stored) this.items = JSON.parse(stored);
            }
        },
        saveToStorage() {
            if (import.meta.client) {
                localStorage.setItem('key', JSON.stringify(this.items));
            }
        },
    },
});
```

## Rules

- Store name convention: `useEntityStore` in file `stores/entity.ts`
- State typed via interface or inline return type
- ALL browser APIs (localStorage, window, document) wrapped in `if (import.meta.client)`
- Getters for derived/computed data (filtering, sorting, searching)
- Actions for async operations and state mutations
- **Error handling**: use `.then().catch().finally()` for async calls (NOT try/catch with await). Use try/catch only for synchronous critical operations
- Unicode-safe search: `.normalize('NFD').replace(/[\u0300-\u036f]/g, '')`
- Conditional loading: check `items.length === 0` before fetching

## Workflow

1. Ask for store name and domain
2. Read related types from `app/types/`
3. Read existing stores for conventions (`Glob stores/*.ts`)
4. Create store in `app/stores/`
5. Create TypeScript interfaces in `app/types/` if needed
