# WhylLima Subagents – Categories

Coleção de agentes especializados para o projeto WhylLima. Cada agente é focado em uma tarefa específica para otimizar uso de tokens e dar suporte preciso ao código.

**Hub:** [github.com/whyllima/whyl-subagents](https://github.com/whyllima/whyl-subagents)

---

## Categorias

### 01-laravel (`whyll-agents`)

Especialistas Laravel 12 para o backend WhylLima. Todos os agentes seguem o padrão Service-Repository com modelos UUID.

**Stack:** PHP 8.3 | Laravel 12 | JWT/Sanctum | Horizon | Reverb | PHPUnit

**Uso:** `@whyll-agents:<agente>`

---

### 02-front-end (`whyll-agents-front`)

Especialistas Nuxt 3 para o front-end. Builders para páginas, componentes, stores, composables, API services, i18n, layouts e plugins.

**Stack:** Nuxt 3 | Vue 3 | Pinia | Axios | PrimeVue | Tailwind CSS | TypeScript | @nuxtjs/i18n

**Uso:** `@whyll-agents-front:<agente>`

---

### 03-business-product (`whyll-agents-biz`)

Especialistas em produto, UX, negócios e conteúdo. Inclui o **Discovery Pipeline:** product-manager → ux-researcher → business-analyst → content-marketer (ideia → problema real → requisitos/viabilidade → posicionamento).

**Detalhes:** [03-business-product/README.md](03-business-product/README.md) | [DISCOVERY-PIPELINE.md](03-business-product/DISCOVERY-PIPELINE.md)

**Uso:** `@whyll-agents-biz:<agente>`

---

## Agentes Front-end (02-front-end)

| Agente | Propósito | Linhas | ~Tokens |
|--------|-----------|--------|---------|
| `nuxt-project-builder` | Projeto Nuxt 3 do zero | 79 | ~200 |
| `page-builder` | Páginas com file-based routing | 53 | ~150 |
| `component-builder` | Componentes Vue SFC | 49 | ~145 |
| `store-builder` | Pinia stores com SSR safety | 77 | ~170 |
| `composable-builder` | Composables reutilizáveis | 55 | ~135 |
| `api-service-builder` | Camada API: Axios → Proxy → Nitro | 87 | ~220 |
| `i18n-manager` | Locales e traduções | 72 | ~150 |
| `layout-builder` | Layouts, Sidebar, Menu, dark mode | 55 | ~155 |
| `plugin-builder` | Plugins Nuxt (client/server) | 52 | ~120 |

### Resumo de Tokens (02-front-end)

| Categoria | Total Linhas | ~Tokens |
|-----------|--------------|---------|
| Builders | 579 | ~1.4k |

---

### Convenções (02-front-end)

| Convenção | Padrão |
|-----------|--------|
| SFC Order | `<script setup>` → `<template>` → `<style scoped>` |
| PrimeVue | Componentes auto-importados, NUNCA importar. Só utilities: `FilterMatchMode`, `useToast`, `useConfirm` |
| Error Handling | `.then().catch()` para async. try/catch só para ops síncronas críticas |
| SSR Safety | `import.meta.client`, `<ClientOnly>`, `typeof window !== 'undefined'` |
| i18n | `useI18n()` + `t()` para todo texto. Estratégia `no_prefix` |
| State | Pinia stores para estado complexo; composables para lógica UI |
| Styling | Tailwind + `:deep()` para PrimeVue overrides |
| Path Alias | `@/` → `app/`, `#imports` para auto-imports Nuxt |

---

### Guia de decisão (02-front-end)

- Projeto novo do zero → `nuxt-project-builder`
- Nova página/rota → `page-builder`
- Novo componente visual → `component-builder`
- Gerenciamento de estado → `store-builder`
- Lógica reutilizável/UI → `composable-builder`
- Integração com API externa → `api-service-builder`
- Traduções/idiomas → `i18n-manager`
- Layout/sidebar/menu/dark mode → `layout-builder`
- Plugin Vue/Nuxt → `plugin-builder`

---

## Arquitetura (01-laravel)

```
Controller (sem lógica) → Service (toda lógica) → Repository (acesso via Model) → Model
                                                                              ↓
Resposta sempre via Resource ←────────────────────────────────────────────────
```

**Regras:**
- **Controller:** só delega para Service, sem try-catch, sem Log, sem queries.
- **Service:** toda lógica de negócio, sempre retorna Resource.
- **Repository:** usa sempre **Model** (nunca facade `DB`).
- **Resource:** todas as respostas da API passam por Resource.

---

## Agentes Laravel – Builders (criar)

| Agente | Propósito | Linhas | ~Tokens |
|--------|-----------|--------|---------|
| `full-stack-specialist` | Feature completa do zero | 197 | ~500 |
| `model-builder` | Só estrutura de dados | 63 | ~160 |
| `api-layer-builder` | API para model existente | 182 | ~450 |
| `endpoint-builder` | Só rotas | 95 | ~240 |
| `test-builder` | Testes PHPUnit | 152 | ~380 |
| `job-builder` | Jobs em fila | 55 | ~140 |
| `resource-builder` | Transformação JSON | 57 | ~140 |
| `seeder-builder` | Dados de teste/dev | 68 | ~170 |
| `event-builder` | Eventos e broadcasting | 79 | ~200 |
| `jwt-auth-builder` | Autenticação JWT completa | 587 | ~1.5k |
| `acl-builder` | ACL Modular (3 níveis) | 723 | ~1.8k |
| `audit-builder` | Auditoria de Models | 421 | ~1k |

---

## Agentes Laravel – Qualidade e Fixers (auditar e corrigir)

| Agente | Propósito | Linhas | ~Tokens |
|--------|-----------|--------|---------|
| `quality-checker` | Audita código e gera prompts | 131 | ~330 |
| `controller-fixer` | Remove lógica do Controller | 88 | ~220 |
| `service-fixer` | Garante Repository e Resources | 75 | ~190 |
| `repository-fixer` | Converte `DB` para Model | 61 | ~150 |
| `model-fixer` | Traits, UUID e relacionamentos | 57 | ~140 |
| `formrequest-fixer` | Validação create/update | 50 | ~125 |
| `resource-fixer` | Campos explícitos, datas ISO | 54 | ~135 |

---

## Resumo de Tokens (01-laravel)

| Categoria | Total Linhas | ~Tokens |
|-----------|--------------|---------|
| Builders | 2.679 | ~6.7k |
| Fixers | 516 | ~1.3k |
| **Total** | **3.195** | **~8k** |

> **Nota:** Estimativa baseada em ~2.5 tokens/linha. Tokens reais podem variar.

---

## Guia de decisão (01-laravel)

**Criar algo novo**
- Feature completa do zero → `full-stack-specialist`
- Só banco de dados → `model-builder`
- Model existe, precisa de API → `api-layer-builder`
- Controller existe, só rotas → `endpoint-builder`
- API existe, precisa de testes → `test-builder`
- Processamento em background → `job-builder`
- Transformação JSON → `resource-builder`
- Dados de teste/dev → `seeder-builder`
- Eventos/real-time → `event-builder`
- Autenticação JWT → `jwt-auth-builder`
- Sistema de permissões modular → `acl-builder`
- Auditoria de alterações → `audit-builder`

**Auditar e corrigir**
- Auditar feature ou arquivos → `quality-checker` (gera prompts para os fixers)
- Corrigir Controller → `controller-fixer`
- Corrigir Service → `service-fixer`
- Corrigir Repository → `repository-fixer`
- Corrigir Model → `model-fixer`
- Corrigir FormRequest → `formrequest-fixer`
- Corrigir Resource → `resource-fixer`

---

## Convenções (01-laravel)

| Convenção | Padrão |
|-----------|--------|
| Primary Key | UUID, coluna `uuid` |
| Foreign Keys | `{entity}_uuid` |
| Traits do Model | HasFactory, HasUuids, SoftDeletes, AutoIncrementId, Auditable |
| Services | Estendem `App\Services\Service` |
| Repositories | Estendem `App\Repositories\Repository` |
| Repository | Nunca usa `DB::`; sempre `Model::query()` |
| Documentação | PHPDoc em inglês, sem comentários inline |
| Rotas | Agrupamento com `Route::controller()` |
| Formatação | `vendor/bin/pint --dirty` |

---

## Exemplos de uso

**Laravel (`whyll-agents`):**
```text
# Criar feature completa
@whyll-agents:full-stack-specialist Create a complete feature for "Categories"

# Configurar autenticação JWT
@whyll-agents:jwt-auth-builder Setup JWT authentication

# Configurar ACL modular (User → Role → Module → Permission)
@whyll-agents:acl-builder Setup modular ACL system

# Configurar auditoria de models
@whyll-agents:audit-builder Setup auditing system

# Verificar configuração de audit nos models
@whyll-agents:audit-builder Check audit configuration in models

# Auditar e obter prompts de correção
@whyll-agents:quality-checker Audit Entity feature

# Corrigir um componente (usar prompt gerado pelo quality-checker)
@whyll-agents:controller-fixer Fix EntityController
File: app/Http/Controllers/EntityController.php
Issues: try-catch blocks, Log:: calls, direct queries
```

**Front-end (`whyll-agents-front`):**
```text
# Criar projeto Nuxt do zero
@whyll-agents-front:nuxt-project-builder Create a Nuxt project called "my-app"

# Criar página
@whyll-agents-front:page-builder Create page /dashboard/analytics

# Criar componente
@whyll-agents-front:component-builder Create UserProfileCard in dashboard domain

# Criar store
@whyll-agents-front:store-builder Create store for managing notifications

# Criar composable
@whyll-agents-front:composable-builder Create useDebounce composable

# Criar camada de API
@whyll-agents-front:api-service-builder Create API service for /users endpoints

# Gerenciar traduções
@whyll-agents-front:i18n-manager Add translations for notifications feature

# Criar layout
@whyll-agents-front:layout-builder Create admin layout with sidebar

# Criar plugin
@whyll-agents-front:plugin-builder Create client-only plugin for analytics
```

**Business & Product (`whyll-agents-biz`):**
```text
# Discovery pipeline: ideia → problema → requisitos → posicionamento
@whyll-agents-biz:product-manager Define product idea and value hypothesis for "X"
@whyll-agents-biz:ux-researcher Validate if user pain is real for "X"
@whyll-agents-biz:business-analyst Translate idea into requirements and viability
@whyll-agents-biz:content-marketer Define positioning and narrative

# Outros agentes
@whyll-agents-biz:project-manager Plan project timeline and resources
@whyll-agents-biz:scrum-master Facilitate sprint planning
```
