# WhylLima Subagents

Specialized AI agents for the WhylLima Laravel project. Each agent is focused on a specific task to optimize token usage and provide precise, context-aware assistance.

---

## Quick Start

### In Claude Code

```text
# Add marketplace
/plugin marketplace add /Users/whyl/.claude/plugins/marketplaces/whyl-subagents

# Install plugin
/plugin install whyll-agents@whyl-subagents

# Use an agent
@whyll-agents:full-stack-specialist Create a complete feature for "Categories"
```

---

## Available Agents

### Core Agents (01-laravel)

| Agent | Purpose | Creates |
|-------|---------|---------|
| `full-stack-specialist` | Complete feature from scratch | Migration, Model, Repository, Service, Form Request, Controller, Routes |
| `model-builder` | Database structure only | Migration, Model |
| `api-layer-builder` | API for existing model | Repository, Service, Form Request, Controller, Routes |
| `endpoint-builder` | Routes only | API Routes |
| `test-builder` | PHPUnit tests | Feature Tests (uses DatabaseTransactions) |
| `job-builder` | Queue jobs | Jobs, Horizon config |
| `resource-builder` | API Resources | JsonResource, ResourceCollection |
| `seeder-builder` | Factories & Seeders | Factories, Seeders, Faker data |
| `event-builder` | Events & Broadcasting | Events, Listeners, Reverb channels |

---

## Agent Details

### full-stack-specialist

**Use when:** Creating a completely new feature from database to API.

**Creates (in order):**
1. Migration (UUID primary key, soft deletes, indexes)
2. Model (HasFactory, HasUuids, SoftDeletes, AutoIncrementId, Auditable)
3. Repository (extends `App\Repositories\Repository`)
4. Service (extends `App\Services\Service`)
5. Form Request (store/update validation with UUID ignore)
6. Controller (CRUD with try-catch and logging)
7. API Routes (`Route::controller()` pattern)

**Example:**
```text
@whyll-agents:full-stack-specialist Create a complete feature for "Categories" with name, slug, description, and status fields.
```

---

### model-builder

**Use when:** You only need database structure, without API layer.

**Creates:**
1. Migration (UUID primary key, soft deletes, indexes)
2. Model (all required traits and relationships)

**Does NOT create:** Repository, Service, Controller, Routes, Tests

**Example:**
```text
@whyll-agents:model-builder Create migration and model for "Tags" with name, slug, color, and entity_uuid (belongsTo Entity).
```

---

### api-layer-builder

**Use when:** Model already exists and you need to expose it via API.

**Prerequisites:** Migration and Model must exist.

**Creates:**
1. Repository
2. Service
3. Form Request
4. Controller
5. API Routes

**Does NOT create:** Migration, Model, Tests

**Example:**
```text
@whyll-agents:api-layer-builder Create API layer for the existing Tag model with CRUD and getByEntity custom action.
```

---

### endpoint-builder

**Use when:** Controller exists and you only need to add routes.

**Prerequisites:** Controller and its methods must exist.

**Creates:** API Routes in `routes/api.php`

**Example:**
```text
@whyll-agents:endpoint-builder Add routes for TagController with methods: index, show, store, update, destroy, getByEntity.
```

---

### test-builder

**Use when:** API endpoints exist and you need test coverage.

**Prerequisites:** Controller, routes, and factory must exist.

**Creates:** Feature Tests (PHPUnit)
- CRUD operations tests
- Validation tests
- Authentication tests (401)
- Custom endpoint tests

**Important:** Uses `DatabaseTransactions` (does NOT wipe database).

**Example:**
```text
@whyll-agents:test-builder Create feature tests for TagController covering all CRUD operations and getByEntity.
```

---

### job-builder

**Use when:** You need background processing with queues.

**Creates:**
1. Queue Jobs (ShouldQueue interface)
2. Horizon configuration
3. Logging setup for `jobs` channel

**Features:**
- Retry logic with backoff
- Failed job handling
- Horizon tags for filtering
- Batch and chain job patterns

**Example:**
```text
@whyll-agents:job-builder Create a job to sync entities with external API, with retry logic and Horizon monitoring.
```

---

### resource-builder

**Use when:** You need to transform Eloquent models to consistent JSON API responses.

**Prerequisites:** Model must exist.

**Creates:**
1. JsonResource (model transformation)
2. ResourceCollection (paginated responses with metadata)

**Features:**
- Conditional relationships with `whenLoaded()`
- Counts with `whenCounted()`
- Computed attributes
- HATEOAS `_links`
- Proper date formatting (ISO 8601)

**Example:**
```text
@whyll-agents:resource-builder Create EntityResource and EntityCollection with category, tags relationships, and comments_count.
```

---

### seeder-builder

**Use when:** You need test/development data with realistic values.

**Prerequisites:** Model and relationships must exist.

**Creates:**
1. Factory (definition + states)
2. Seeder (data population)

**Features:**
- Faker for realistic data
- States (published, draft, featured, archived)
- Relationship handling (recycle, afterCreating)
- Predefined data seeders

**Example:**
```text
@whyll-agents:seeder-builder Create EntityFactory with states (published, draft, featured) and EntitySeeder with 50 sample records.
```

---

### event-builder

**Use when:** You need domain events, listeners, or real-time broadcasting.

**Creates:**
1. Events (standard and broadcastable)
2. Listeners (sync and queued)
3. Channel authorization (`routes/channels.php`)
4. Event Subscribers

**Features:**
- Laravel Reverb integration
- Private, Public, and Presence channels
- Conditional broadcasting (`broadcastWhen`)
- Custom event names (`broadcastAs`)

**Example:**
```text
@whyll-agents:event-builder Create EntityCreated, EntityUpdated events with broadcasting to private channel and notification listener.
```

---

## Decision Guide

```
Need a new feature?
│
├── From scratch (no model exists)
│   └── Use: full-stack-specialist
│
├── Only database structure needed
│   └── Use: model-builder
│
├── Model exists, need API
│   └── Use: api-layer-builder
│
├── Controller exists, need routes
│   └── Use: endpoint-builder
│
├── API exists, need tests
│   └── Use: test-builder
│
├── Need background processing
│   └── Use: job-builder
│
├── Need JSON transformation
│   └── Use: resource-builder
│
├── Need test/dev data
│   └── Use: seeder-builder
│
└── Need real-time/events
    └── Use: event-builder
```

---

## Project Stack

- **PHP:** 8.3.27
- **Laravel:** v12
- **Auth:** Sanctum v4
- **Queue:** Horizon v5
- **Real-time:** Reverb v1
- **Performance:** Octane v2
- **Code Style:** Pint v1
- **Testing:** PHPUnit v11

---

## Architecture

```
Controller → Service → Repository → Model
```

### Key Conventions

| Convention | Pattern |
|-----------|---------|
| Primary Key | UUID with column name `uuid` |
| Foreign Keys | `{entity}_uuid` naming |
| Model Traits | `HasFactory`, `HasUuids`, `SoftDeletes`, `AutoIncrementId`, `Auditable` |
| Services | Extend `App\Services\Service` |
| Repositories | Extend `App\Repositories\Repository` |
| Documentation | PHPDoc in English, no inline comments |
| Routes | `Route::controller()` grouping |
| Code Format | `vendor/bin/pint --dirty` |

---

## Token Optimization

| Scenario | Recommended Agent | Token Cost |
|----------|-------------------|------------|
| Complete new feature | `full-stack-specialist` | High (~694 lines) |
| Just database | `model-builder` | Low (~229 lines) |
| API for existing model | `api-layer-builder` | Medium (~457 lines) |
| Just routes | `endpoint-builder` | Low (~277 lines) |
| Just tests | `test-builder` | Low (~167 lines) |
| Just jobs | `job-builder` | Low (~139 lines) |
| JSON transformation | `resource-builder` | Low (~280 lines) |
| Test/dev data | `seeder-builder` | Medium (~400 lines) |
| Events/real-time | `event-builder` | Medium (~450 lines) |

**Tip:** Chain smaller agents instead of using full-stack-specialist:
```
model-builder → api-layer-builder → test-builder
```

---

## Future Agents (Planned)

| Agent | Purpose | Status |
|-------|---------|--------|
| `policy-builder` | Policies and Gates (authorization) | Planned |
| `observer-builder` | Model Observers (lifecycle hooks) | Planned |
| `notification-builder` | Mail, SMS, Database notifications | Planned |
| `command-builder` | Artisan commands | Planned |
| `middleware-builder` | Custom middleware | Planned |

---

## File Structure

```
whyl-subagents/
├── README.md                          ← This file
├── .claude/
│   └── settings.local.json            ← Permissions (pint, artisan, composer)
├── .claude-plugin/
│   └── marketplace.json               ← Marketplace definition
└── categories/
    ├── README.md                      ← Category overview
    └── 01-laravel/
        ├── .claude-plugin/
        │   └── plugin.json            ← Plugin definition (whyll-agents)
        ├── full-stack-specialist.md
        ├── model-builder.md
        ├── api-layer-builder.md
        ├── endpoint-builder.md
        ├── test-builder.md
        ├── job-builder.md
        ├── resource-builder.md
        ├── seeder-builder.md
        └── event-builder.md
```

---

## Commands Reference

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

# Cache
php artisan config:cache
php artisan route:cache

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
