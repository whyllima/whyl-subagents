# WhylLima Subagents

Collection of specialized AI agents for the WhylLima project. Each agent is focused on a specific task to optimize token usage and provide precise, context-aware assistance.

---

## Categories

### 01-laravel

Laravel 12 specialists for the WhylLima backend project. All agents follow the Service-Repository pattern with UUID-based models.

**Stack:** PHP 8.3 | Laravel 12 | Sanctum | Horizon | PHPUnit

---

## Agents Overview

| Agent | Purpose | Creates |
|-------|---------|---------|
| full-stack-specialist | Complete feature from scratch | Migration, Model, Repository, Service, Form Request, Controller, Routes |
| model-builder | Database structure only | Migration, Model |
| api-layer-builder | API for existing model | Repository, Service, Form Request, Controller, Routes |
| endpoint-builder | Routes only | API Routes |
| test-builder | PHPUnit tests | Feature Tests (DatabaseTransactions) |
| job-builder | Queue jobs | Jobs, Horizon config |

---

## Decision Guide

- **New feature from scratch** → full-stack-specialist
- **Only database structure** → model-builder
- **Model exists, need API** → api-layer-builder
- **Controller exists, need routes** → endpoint-builder
- **API exists, need tests** → test-builder
- **Background processing** → job-builder

---

## Conventions

- UUID as primary key with column name `uuid`
- Foreign keys: `{entity}_uuid` naming
- Required Model traits: HasFactory, HasUuids, SoftDeletes, AutoIncrementId, Auditable
- Services extend `App\Services\Service`
- Repositories extend `App\Repositories\Repository`
- PHPDoc in English, no inline comments
- Route pattern: `Route::controller()` grouping
- Code formatting: `vendor/bin/pint --dirty`
