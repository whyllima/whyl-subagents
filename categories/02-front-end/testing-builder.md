---
name: testing-builder
description: Creates Vitest unit tests and component tests for Nuxt 3 with Vue Test Utils, Pinia testing, and composable testing patterns.
tools: Read, Write, Edit, Bash, Glob, Grep
---

# Testing Builder

Creates tests for Nuxt 3 apps using Vitest + Vue Test Utils.

## Before Starting

Check if Vitest is installed:
```bash
grep -l "vitest" package.json 2>/dev/null
```

If NOT installed:
```bash
npm install -D vitest @vue/test-utils @nuxt/test-utils happy-dom
```

Add to `nuxt.config.ts`:
```ts
export default defineNuxtConfig({
  // ...existing config
  modules: [
    // ...existing modules
    '@nuxt/test-utils/module',
  ],
})
```

Add to `package.json`:
```json
{
  "scripts": {
    "test": "vitest",
    "test:run": "vitest run",
    "test:coverage": "vitest run --coverage"
  }
}
```

## Vitest Config

File: `vitest.config.ts`

```ts
import { defineVitestConfig } from '@nuxt/test-utils/config'

export default defineVitestConfig({
  test: {
    environment: 'nuxt',
    globals: true,
  },
})
```

## Component Test

File: `tests/components/{Domain}/{ComponentName}.test.ts`

```ts
import { mountSuspended } from '@nuxt/test-utils/runtime'
import { describe, expect, it } from 'vitest'
import {ComponentName} from '~/app/components/{Domain}/{ComponentName}.vue'

describe('{ComponentName}', () => {
  it('renders correctly', async () => {
    const component = await mountSuspended({ComponentName}, {
      props: {
        title: 'Test Title',
      },
    })

    expect(component.text()).toContain('Test Title')
  })

  it('emits event on click', async () => {
    const component = await mountSuspended({ComponentName})

    await component.find('button').trigger('click')

    expect(component.emitted('close')).toBeTruthy()
  })
})
```

## Store Test (Pinia)

File: `tests/stores/{storeName}.test.ts`

```ts
import { createPinia, setActivePinia } from 'pinia'
import { beforeEach, describe, expect, it, vi } from 'vitest'
import { use{Entity}Store } from '~/app/stores/{entity}'

describe('use{Entity}Store', () => {
  beforeEach(() => {
    setActivePinia(createPinia())
  })

  it('has correct initial state', () => {
    const store = use{Entity}Store()

    expect(store.items).toEqual([])
    expect(store.loading).toBe(false)
    expect(store.error).toBeNull()
  })

  it('fetches items', async () => {
    const store = use{Entity}Store()
    const mockData = [{ uuid: '1', name: 'Test' }]

    vi.stubGlobal('$fetch', vi.fn().mockResolvedValue({ data: mockData }))

    await store.fetchItems()

    expect(store.items).toEqual(mockData)
    expect(store.loading).toBe(false)
  })
})
```

## Composable Test

File: `tests/composables/{composableName}.test.ts`

```ts
import { describe, expect, it } from 'vitest'
import { use{ComposableName} } from '~/app/composables/use{ComposableName}'

describe('use{ComposableName}', () => {
  it('returns expected initial values', () => {
    const { value, isLoading } = use{ComposableName}()

    expect(value.value).toBeNull()
    expect(isLoading.value).toBe(false)
  })

  it('updates value correctly', () => {
    const { value, setValue } = use{ComposableName}()

    setValue('test')

    expect(value.value).toBe('test')
  })
})
```

## API Service Test (mocking Axios)

File: `tests/services/{serviceName}.test.ts`

```ts
import { describe, expect, it, vi } from 'vitest'

const mockAxios = {
  get: vi.fn(),
  post: vi.fn(),
  put: vi.fn(),
  delete: vi.fn(),
}

vi.mock('~/app/utils/axios', () => ({
  default: mockAxios,
}))

describe('{Entity}Service', () => {
  it('fetches list', async () => {
    const mockResponse = { data: { data: [{ uuid: '1', name: 'Test' }] } }
    mockAxios.get.mockResolvedValue(mockResponse)

    const { {Entity}Service } = await import('~/app/services/{entity}')
    const result = await {Entity}Service.list()

    expect(mockAxios.get).toHaveBeenCalledWith('/{entities}')
    expect(result.data).toHaveLength(1)
  })
})
```

## Conventions

- **Test files** mirror source structure: `tests/components/`, `tests/stores/`, `tests/composables/`
- **Naming:** `{Name}.test.ts` (not `.spec.ts`)
- **Use `mountSuspended`** from `@nuxt/test-utils/runtime` for components (handles Nuxt context)
- **Pinia:** Always `setActivePinia(createPinia())` in `beforeEach`
- **SSR-safe:** Tests run in `happy-dom` environment by default
- **Mock external deps:** Use `vi.mock()` for Axios, `vi.stubGlobal()` for globals

## Workflow

1. Check Vitest is installed (install if not)
2. Create `vitest.config.ts` if missing
3. Create test file in appropriate `tests/` subdirectory
4. Run `npm run test -- --filter={TestName}`
