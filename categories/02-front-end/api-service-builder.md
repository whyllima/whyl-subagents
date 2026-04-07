---
name: api-service-builder
description: Creates Nuxt 4 API service layers with Axios client, Nitro server proxy, and secure server-side API calls.
tools: Read, Write, Edit, Bash, Glob, Grep
---

# API Service Builder

Creates: Client Axios utils, Server proxy handlers, Service classes.

## Architecture

```
Browser (Axios) → /api/proxy/* → Server Nitro → External API
     client-side     server-side      (API_BASE_URL)
```

## Layer 1 - Axios Client (`app/utils/axios.ts`)

```typescript
import axios from 'axios';

const instance = axios.create({
    baseURL: '/api/proxy',
    timeout: 30000,
    headers: { 'Content-Type': 'application/json', Accept: 'application/json' },
});

// Request interceptor: add Bearer token
// Response interceptor: handle 401 (clear token + redirect), 403, 404, 500
// Network error detection: error.request && !error.response

export const api = instance;
```

## Layer 2 - Server Proxy (`server/api/proxy/[...path].ts`)

```typescript
export default defineEventHandler(async (event) => {
    const path = event.context.params?.path || '';
    const method = getMethod(event);
    const query = getQuery(event);
    let body = null;
    if (['POST', 'PUT', 'PATCH'].includes(method)) body = await readBody(event);
    return await secureApiCall(`/${path}`, { method, query, body });
});
```

## Layer 3 - Secure Call (`server/utils/api.ts`)

```typescript
export async function secureApiCall(endpoint: string, options: any = {}) {
    const config = useRuntimeConfig();
    const headers = { 'Content-Type': 'application/json', Accept: 'application/json', ...options.headers };
    if (config.apiToken) headers.Authorization = `Bearer ${config.apiToken}`;
    return await $fetch(`${config.apiBaseUrl}${endpoint}`, { ...options, headers });
}
```

Import within server code using `#server` alias:
```typescript
import { secureApiCall } from '#server/utils/api';
```

## Layer 4 - Service Class (`app/service/`)

For local/static data (not external API):

```typescript
export class EntityService {
    static async loadData(): Promise<Entity[]> {
        const response = await fetch('/data/entities.json'); // native fetch for static files
        return await response.json();
    }
}
```

## Alternative: createUseFetch (Nuxt 4)

Instead of Layer 1 Axios, use Nuxt 4's `createUseFetch` for SSR-compatible data fetching:

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

// Usage in pages/components:
const { data: items, status } = await useApi('/items');
```

## Error Handling (Nuxt 4)

In server handlers, use updated Web API naming:

```typescript
// Nuxt 4: use status/statusText (not statusCode/statusMessage)
throw createError({ status: 404, statusText: 'Not Found' });
throw createError({ status: 422, statusText: 'Validation failed', data: errors });
```

## Rules

- External API calls: always through Axios → Proxy → secureApiCall (never expose API_TOKEN)
- Static data: native `fetch()` directly (no proxy needed)
- Token management: `localStorage`/`sessionStorage` with `import.meta.client` guard
- 401 handling: clear tokens + redirect to `/auth/login`
- runtimeConfig: `apiBaseUrl` and `apiToken` server-only; `public.appName` client-accessible
- **Error handling**: use `.then().catch().finally()` for async API calls (NOT try/catch with await). Use try/catch only for synchronous critical operations (e.g. JSON.parse)
- **Nuxt 4**: Prefer `createUseFetch` over raw Axios for SSR-compatible pages; keep Axios for client-only or plugin contexts

## Workflow

1. Determine need: external API proxy, createUseFetch composable, service class, or Axios endpoint
2. Read existing `server/api/`, `app/utils/`, `app/service/`, `app/composables/`
3. Create appropriate layer files
4. Add runtimeConfig entries to `nuxt.config.ts` if needed
