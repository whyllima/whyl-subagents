---
name: middleware-builder
description: Creates Nuxt 4 route middleware for authentication, authorization, redirects, and guards with SSR safety.
tools: Read, Write, Edit, Bash, Glob, Grep
---

# Middleware Builder

Creates Nuxt 4 route middleware for auth guards, redirects, and access control.

## Middleware Types

| Type | Location | Scope |
|------|----------|-------|
| Named | `app/middleware/{name}.ts` | Applied per-page via `definePageMeta` |
| Global | `app/middleware/{name}.global.ts` | Runs on every route change |

## Named Middleware — Auth Guard

File: `app/middleware/auth.ts`

```ts
export default defineNuxtRouteMiddleware((to, from) => {
  const token = useCookie('token')

  if (!token.value) {
    return navigateTo('/auth/login')
  }
})
```

## Named Middleware — Guest Only (redirect if logged in)

File: `app/middleware/guest.ts`

```ts
export default defineNuxtRouteMiddleware((to, from) => {
  const token = useCookie('token')

  if (token.value) {
    return navigateTo('/dashboard')
  }
})
```

## Named Middleware — Role Guard

File: `app/middleware/role.ts`

```ts
export default defineNuxtRouteMiddleware((to, from) => {
  const authStore = useAuthStore()
  const requiredRole = to.meta.requiredRole as string | undefined

  if (requiredRole && !authStore.hasRole(requiredRole)) {
    return navigateTo('/unauthorized')
  }
})
```

## Named Middleware — Permission Guard

File: `app/middleware/permission.ts`

```ts
export default defineNuxtRouteMiddleware((to, from) => {
  const authStore = useAuthStore()
  const requiredPermission = to.meta.requiredPermission as string | undefined

  if (requiredPermission && !authStore.hasPermission(requiredPermission)) {
    return navigateTo('/unauthorized')
  }
})
```

## Global Middleware — Token Refresh

File: `app/middleware/token-refresh.global.ts`

```ts
export default defineNuxtRouteMiddleware(async (to, from) => {
  if (import.meta.server) return

  const token = useCookie('token')
  const refreshToken = useCookie('refresh_token')

  if (!token.value && refreshToken.value) {
    const { $axios } = useNuxtApp()
    $axios.post('/auth', {}, {
      headers: { Authorization: `Bearer ${refreshToken.value}` }
    }).then((res: any) => {
      token.value = res.data.access_token
    }).catch(() => {
      refreshToken.value = null
      navigateTo('/auth/login')
    })
  }
})
```

## Route Groups (Nuxt 4)

Use parenthesized folders for convention-based authorization:

```
app/pages/(admin)/dashboard.vue  → meta.groups includes 'admin'
app/pages/(admin)/users.vue      → meta.groups includes 'admin'
app/pages/(public)/about.vue     → meta.groups includes 'public'
```

Global middleware can check route groups:

```ts
// app/middleware/admin.global.ts
export default defineNuxtRouteMiddleware((to) => {
  if (to.meta.groups?.includes('admin')) {
    const authStore = useAuthStore()
    if (!authStore.hasRole('admin')) {
      return navigateTo('/unauthorized')
    }
  }
})
```

## Using Middleware in Pages

```ts
// app/pages/dashboard.vue
definePageMeta({
  middleware: ['auth'],
  layout: 'admin',
})
```

```ts
// app/pages/admin/users.vue
definePageMeta({
  middleware: ['auth', 'role'],
  requiredRole: 'admin',
})
```

```ts
// app/pages/auth/login.vue
definePageMeta({
  middleware: ['guest'],
  layout: 'blank',
})
```

## SSR Safety Rules

- Use `useCookie()` for token access (works server + client)
- NEVER use `localStorage` in middleware (server-side incompatible)
- Use `import.meta.server` guard for server-only logic
- Use `navigateTo()` for redirects (not `window.location`)

## Workflow

1. Determine middleware type (named vs global)
2. Consider route groups for convention-based authorization
3. Create middleware in `app/middleware/`
4. Apply via `definePageMeta` in target pages (named middleware)
5. Test SSR + client navigation
