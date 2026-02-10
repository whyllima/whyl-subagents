---
name: api-service-builder
description: Creates Nuxt 3 API service layers with Axios client, Nitro server proxy, and secure server-side API calls.
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

## Rules

- External API calls: always through Axios → Proxy → secureApiCall (never expose API_TOKEN)
- Static data: native `fetch()` directly (no proxy needed)
- Token management: `localStorage`/`sessionStorage` with `import.meta.client` guard
- 401 handling: clear tokens + redirect to `/auth/login`
- runtimeConfig: `apiBaseUrl` and `apiToken` server-only; `public.appName` client-accessible
- **Error handling**: use `.then().catch().finally()` for async API calls (NOT try/catch with await). Use try/catch only for synchronous critical operations (e.g. JSON.parse)

## Workflow

1. Determine need: external API proxy, service class, or new Axios endpoint
2. Read existing `server/api/`, `app/utils/`, `app/service/`
3. Create appropriate layer files
4. Add runtimeConfig entries to `nuxt.config.ts` if needed
