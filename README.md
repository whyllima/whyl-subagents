# WhylLima Subagents

Agentes de IA especializados para o projeto WhylLima (Laravel). Cada agente é focado em uma tarefa específica para otimizar uso de tokens e dar suporte preciso ao código.

---

## Quick Start

### No Claude Code

```text
# Adicionar marketplace
/plugin marketplace add /Users/whyl/.claude/plugins/marketplaces/whyl-subagents

# Instalar plugin
/plugin install whyll-agents@whyl-subagents

# Usar um agente
@whyll-agents:full-stack-specialist Create a complete feature for "Categories"
```

---

## Agentes disponíveis

### Builders (criar componentes)

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
| `acl-builder` | Sistema ACL Modular (3 níveis) | 723 | ~1.8k |
| `audit-builder` | Auditoria de Models | 421 | ~1k |

### Qualidade e Fixers (auditar e corrigir)

| Agente | Propósito | Linhas | ~Tokens |
|--------|-----------|--------|---------|
| `quality-checker` | Audita código e gera prompts | 131 | ~330 |
| `controller-fixer` | Remove lógica do Controller | 88 | ~220 |
| `service-fixer` | Garante Repository e Resources | 75 | ~190 |
| `repository-fixer` | Converte `DB::` para Model | 61 | ~150 |
| `model-fixer` | Traits, UUID e relacionamentos | 57 | ~140 |
| `formrequest-fixer` | Validação create/update | 50 | ~125 |
| `resource-fixer` | Campos explícitos, datas ISO | 54 | ~135 |

### Resumo de Tokens

| Categoria | Agentes | Total Linhas | ~Tokens |
|-----------|---------|--------------|---------|
| Builders | 12 | 2.679 | ~6.7k |
| Fixers | 7 | 516 | ~1.3k |
| **Total** | **19** | **3.195** | **~8k** |

> **Nota:** Estimativa baseada em ~2.5 tokens/linha de markdown/código.

---

## Arquitetura

```
Controller (sem lógica) → Service (toda lógica) → Repository (acesso via Model) → Model
                                                                              ↓
Resposta sempre via Resource ←────────────────────────────────────────────────
```

**Regras:**
- **Controller:** só delega para Service; sem try-catch, sem Log, sem queries.
- **Service:** toda lógica de negócio; sempre retorna Resource.
- **Repository:** usa sempre **Model** (nunca facade `DB`).
- **Resource:** todas as respostas da API passam por Resource.

---

## Detalhes dos agentes

### full-stack-specialist

**Quando usar:** Criar uma feature nova do zero (banco + API).

**Cria (nessa ordem):**
1. Migration (UUID como PK, soft deletes, índices)
2. Model (HasFactory, HasUuids, SoftDeletes, AutoIncrementId, Auditable)
3. Repository (extends `App\Repositories\Repository`, queries via Model)
4. Service (extends `App\Services\Service`, retorna Resources)
5. Form Request (validação store/update com UUID ignore)
6. Controller (só delega para Service, sem lógica)
7. Resource e Collection
8. API Routes (`Route::controller()`)

**Exemplo:**
```text
@whyll-agents:full-stack-specialist Create a complete feature for "Categories" with name, slug, description, and status.
```

---

### model-builder

**Quando usar:** Só estrutura de banco, sem camada de API.

**Cria:** Migration, Model (traits e relacionamentos).

**Não cria:** Repository, Service, Controller, Routes, Tests.

**Exemplo:**
```text
@whyll-agents:model-builder Create migration and model for "Tags" with name, slug, color, and entity_uuid (belongsTo Entity).
```

---

### api-layer-builder

**Quando usar:** Model já existe e precisa expor via API.

**Pré-requisitos:** Migration e Model existentes.

**Cria:** Repository, Service, Form Request, Controller, Resource, Collection, Routes.

**Não cria:** Migration, Model, Tests.

**Exemplo:**
```text
@whyll-agents:api-layer-builder Create API layer for the existing Tag model with CRUD and getByEntity custom action.
```

---

### endpoint-builder

**Quando usar:** Controller existe e só falta definir rotas.

**Pré-requisitos:** Controller e métodos existentes.

**Cria:** Rotas em `routes/api.php`.

**Exemplo:**
```text
@whyll-agents:endpoint-builder Add routes for TagController with methods: index, show, store, update, destroy, getByEntity.
```

---

### test-builder

**Quando usar:** Endpoints existem e precisa de testes.

**Pré-requisitos:** Controller, rotas e factory existentes.

**Cria:** Feature Tests (PHPUnit) – CRUD, validação, auth (401).

**Importante:** Usa `DatabaseTransactions` (não limpa o banco).

**Exemplo:**
```text
@whyll-agents:test-builder Create feature tests for TagController covering all CRUD operations and getByEntity.
```

---

### job-builder

**Quando usar:** Processamento em background com filas.

**Cria:** Jobs (ShouldQueue), config Horizon, logging no canal `jobs`.

**Exemplo:**
```text
@whyll-agents:job-builder Create a job to sync entities with external API, with retry logic and Horizon monitoring.
```

---

### resource-builder

**Quando usar:** Transformar models em respostas JSON padronizadas.

**Pré-requisitos:** Model existente.

**Cria:** JsonResource, ResourceCollection (com whenLoaded, whenCounted, datas ISO).

**Exemplo:**
```text
@whyll-agents:resource-builder Create EntityResource and EntityCollection with category, tags relationships, and comments_count.
```

---

### seeder-builder

**Quando usar:** Dados de teste ou desenvolvimento.

**Pré-requisitos:** Model e relacionamentos existentes.

**Cria:** Factory (definition + states), Seeder.

**Exemplo:**
```text
@whyll-agents:seeder-builder Create EntityFactory with states (published, draft, featured) and EntitySeeder with 50 sample records.
```

---

### event-builder

**Quando usar:** Eventos de domínio, listeners ou broadcasting em tempo real.

**Cria:** Events (normais e broadcastable), Listeners (sync e queued), autorização de canais (`routes/channels.php`).

**Exemplo:**
```text
@whyll-agents:event-builder Create EntityCreated, EntityUpdated events with broadcasting to private channel and notification listener.
```

---

### jwt-auth-builder

**Quando usar:** Configurar autenticação JWT com single-session, refresh token, password reset.

**Cria:** AuthController, AuthService, AuthRepository, JwtMiddleware, FormRequests, Resources.

**Detecta ACL:** Verifica se existe ACL Modular ou Simple para incluir roles/permissions no `me()`.

**Exemplo:**
```text
@whyll-agents:jwt-auth-builder Setup JWT authentication with password reset and profile update
```

---

### acl-builder

**Quando usar:** Configurar sistema de permissões modular com três níveis: `User → Role → Module → Permission`.

**Arquitetura:**
```
User ──has── Role ──has── Module ──has── Permission
              │             │
              └─────────────┘
         role_modules_permissions
```

**Cria:**
- Migrations (modules, model_has_permissions, role_modules_permissions)
- Models (Module, ModelHasPermission, RoleModulesPermission, Role, Permission)
- Traits (HasNameLookup, HasModulePermission)
- Middleware (ModelAndPermissionMiddleware)
- Exception (NotLoggedInException)
- Seeders (PermissionSeeder, ModuleSeeder, RoleSeeder)

**Formato de permissão:** `{action}-{module}` (ex: `show-rule`, `index-user`)

**Uso em Controller:**
```php
$this->middleware('model_permission:show-entity')->only(['show']);
$this->middleware('model_permission:index-entity')->only(['index']);
```

**Exemplo:**
```text
@whyll-agents:acl-builder Setup modular ACL system with modules: user, role, entity, campaign
```

---

### audit-builder

**Quando usar:** Configurar sistema de auditoria ou verificar se models estão configurados para audit.

**Cria:**
- Config `audit.php` customizado
- Migration `audits` com UUID
- Resolvers (IpAddress, UserAgent, Url)
- Resources (AuditResource, AuditCollection)

**Também verifica:**
- Models sem interface `ContractAuditable`
- Models sem trait `Auditable`
- Models com password sem `$auditExclude`

**Exemplos:**
```text
# Setup completo
@whyll-agents:audit-builder Setup auditing system

# Verificar models
@whyll-agents:audit-builder Check audit configuration in all models
```

---

### quality-checker

**Quando usar:** Auditar uma feature ou arquivos para ver se seguem o padrão.

**Faz:** Escaneia controllers, services, repositories, models, form requests e resources; identifica violações; **gera prompts prontos** para copiar e usar com os fixers.

**Exemplo:**
```text
@whyll-agents:quality-checker Audit Entity feature
```

**Saída típica:** Lista de componentes com problemas + prompts no formato:
```text
@whyll-agents:controller-fixer Fix EntityController
File: app/Http/Controllers/EntityController.php
Issues: try-catch blocks, Log:: calls, direct queries
```

---

### Fixers (controller-fixer, service-fixer, repository-fixer, model-fixer, formrequest-fixer, resource-fixer)

**Quando usar:** Depois do quality-checker ou quando souber qual arquivo está fora do padrão.

**Uso:** Cole o prompt gerado pelo quality-checker (ou monte um semelhante com arquivo e issues).

**Exemplos:**
```text
@whyll-agents:controller-fixer Fix EntityController - remove logic, delegate to Service. File: app/Http/Controllers/EntityController.php

@whyll-agents:repository-fixer Fix EntityRepository - convert DB facade to Model queries. File: app/Repositories/EntityRepository.php
```

---

## Guia de decisão

```
Precisa criar algo?
│
├── Feature nova do zero
│   └── full-stack-specialist
├── Só banco de dados
│   └── model-builder
├── Model existe, precisa de API
│   └── api-layer-builder
├── Controller existe, só rotas
│   └── endpoint-builder
├── API existe, precisa de testes
│   └── test-builder
├── Processamento em background
│   └── job-builder
├── Transformação JSON
│   └── resource-builder
├── Dados de teste/dev
│   └── seeder-builder
├── Eventos/real-time
│   └── event-builder
├── Autenticação JWT
│   └── jwt-auth-builder
├── Sistema de permissões modular
│   └── acl-builder
└── Auditoria de models
    └── audit-builder

Precisa auditar/corrigir?
│
├── Auditar feature ou arquivos
│   └── quality-checker (gera prompts para os fixers)
└── Corrigir componente específico
    └── controller-fixer | service-fixer | repository-fixer
        | model-fixer | formrequest-fixer | resource-fixer
```

---

## Stack do projeto

- **PHP:** 8.3.27
- **Laravel:** v12
- **Auth:** JWT (tymon/jwt-auth) ou Sanctum v4
- **Queue:** Horizon v5
- **Real-time:** Reverb v1
- **Performance:** Octane v2
- **Code Style:** Pint v1
- **Testing:** PHPUnit v11
- **Auditing:** owen-it/laravel-auditing

---

## Convenções

| Convenção | Padrão |
|-----------|--------|
| Primary Key | UUID, coluna `uuid` |
| Foreign Keys | `{entity}_uuid` |
| Traits do Model | HasFactory, HasUuids, SoftDeletes, AutoIncrementId, Auditable |
| Services | Estendem `App\Services\Service` |
| Repositories | Estendem `App\Repositories\Repository`, **nunca** usam `DB::` |
| Controller | Sem lógica; só delega para Service |
| Respostas | Sempre via Resource |
| Documentação | PHPDoc em inglês, sem comentários inline |
| Rotas | Agrupamento com `Route::controller()` |
| Formatação | `vendor/bin/pint --dirty` |

---

## Estrutura de arquivos

```
whyl-subagents/
├── README.md
├── .claude/
│   └── settings.local.json
├── .claude-plugin/
│   └── marketplace.json
└── categories/
    ├── README.md
    └── 01-laravel/
        ├── .claude-plugin/
        │   └── plugin.json
        ├── full-stack-specialist.md
        ├── model-builder.md
        ├── api-layer-builder.md
        ├── endpoint-builder.md
        ├── test-builder.md
        ├── job-builder.md
        ├── resource-builder.md
        ├── seeder-builder.md
        ├── event-builder.md
        ├── jwt-auth-builder.md
        ├── acl-builder.md
        ├── audit-builder.md
        ├── quality-checker.md
        ├── controller-fixer.md
        ├── service-fixer.md
        ├── repository-fixer.md
        ├── model-fixer.md
        ├── formrequest-fixer.md
        └── resource-fixer.md
```

---

## Comandos úteis

```bash
# Laravel
php artisan make:migration create_entities_table --no-interaction
php artisan make:model Entity -fs --no-interaction
php artisan make:controller EntityController --api --no-interaction
php artisan make:request EntityRequest --no-interaction
php artisan make:job ProcessEntityJob --no-interaction
php artisan make:test EntityControllerTest --no-interaction
php artisan make:resource EntityResource --no-interaction
php artisan make:resource EntityCollection --collection --no-interaction
php artisan make:factory EntityFactory --model=Entity --no-interaction
php artisan make:seeder EntitySeeder --no-interaction
php artisan make:event EntityCreated --no-interaction
php artisan make:listener SendEntityNotification --event=EntityCreated --no-interaction

# Code Style
vendor/bin/pint --dirty

# Testing
php artisan test --filter=EntityControllerTest
php artisan test --parallel

# Queue
php artisan horizon
php artisan horizon:status

# Reverb (WebSockets)
php artisan reverb:start
php artisan reverb:start --debug

# Seeding
php artisan db:seed
php artisan db:seed --class=EntitySeeder
```

---

## License

MIT
