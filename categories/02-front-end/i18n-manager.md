---
name: i18n-manager
description: Creates and manages i18n locale files, translation keys, and @nuxtjs/i18n configuration for Nuxt 3 projects.
tools: Read, Write, Edit, Bash, Glob, Grep
---

# i18n Manager

Manages: Locale files (`locales/`), translation keys, and `@nuxtjs/i18n` config.

## Configuration

```typescript
// nuxt.config.ts
i18n: {
    locales: [
        { code: 'pt', iso: 'pt-BR', file: 'pt.json', name: 'Português' },
        { code: 'en', iso: 'en-US', file: 'en.json', name: 'English' },
        { code: 'es', iso: 'es-ES', file: 'es.json', name: 'Español' },
    ],
    defaultLocale: 'pt',
    langDir: 'locales',
    strategy: 'no_prefix',   // No /pt, /en in URLs
    detectBrowserLanguage: { useCookie: true, cookieKey: 'i18n_redirected' },
}
```

## Locale File Structure

Nested by feature domain:

```json
{
    "featureName": {
        "sectionName": {
            "title": "Translated Title",
            "searchPlaceholder": "Search...",
            "filters": {
                "all": "All",
                "active": "Active"
            }
        }
    }
}
```

## Usage

```typescript
const { t } = useI18n();

// In script
const label = t('featureName.sectionName.title');

// In template
{{ t('featureName.sectionName.filters.all') }}
```

## Rules

- Keys organized by feature domain (nested objects)
- ALL user-facing text must use `t()` — no hardcoded strings in templates
- Keep all locale files in sync (same keys in pt.json, en.json, es.json)
- Strategy `no_prefix`: no language prefix in URLs
- Use `useI18n()` from `#imports` (auto-imported)

## Workflow

1. Read existing locale files (`locales/*.json`)
2. Add/update keys in ALL locale files simultaneously
3. Verify keys match across all languages
4. Use `Grep` to find hardcoded strings in components that should use i18n
