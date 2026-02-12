# Laravel Agents (`whyll-agents`)

Agentes especializados para Laravel 12 seguindo o padrao Service-Repository com UUID.

**Stack:** PHP 8.3 | Laravel 12 | JWT/Sanctum | Horizon | Reverb | PHPUnit

**Prefixo:** `@whyll-agents:<agente>`

---

## Arquitetura

```
Controller (sem logica) -> Service (toda logica) -> Repository (via Model) -> Model
Resposta sempre via Resource
```

---

## Builders (criar do zero)

### `full-stack-specialist`

Cria uma feature completa de ponta a ponta: Migration, Model, Repository, Service, FormRequest, Controller, Resource e Routes. Detecta automaticamente se o projeto usa ACL e adiciona middleware de permissoes.

**Exemplos:**

```text
@whyll-agents:full-stack-specialist Create a complete feature for "Products" with name, description, price, and category relationship

@whyll-agents:full-stack-specialist Create a complete feature for "Orders" with status, total, user relationship, and order items

@whyll-agents:full-stack-specialist Create a complete CRUD for "Tags" with name and slug fields
```

---

### `model-builder`

Cria Migration e Model com UUID como primary key, traits obrigatorias (HasFactory, HasUuids, SoftDeletes, AutoIncrementId, Auditable), e relacionamentos com chaves explicitas. Tambem cria tabelas pivot quando necessario.

**Exemplos:**

```text
@whyll-agents:model-builder Create migration and model for "Article" with title, body, status, and belongsTo Category

@whyll-agents:model-builder Create migration and model for "Invoice" with number, total, due_date, and belongsTo Customer

@whyll-agents:model-builder Create pivot table and belongsToMany relationship between Product and Tag
```

---

### `api-layer-builder`

Cria a camada de API completa para um Model ja existente: Repository, Service, FormRequest, Controller, Resource, Collection e Routes. Detecta ACL e adiciona middleware quando presente.

**Exemplos:**

```text
@whyll-agents:api-layer-builder Create API layer for existing Product model

@whyll-agents:api-layer-builder Create full API (Repository, Service, Controller, Resource, Routes) for the Article model

@whyll-agents:api-layer-builder Create API endpoints with filters for the Invoice model (filter by status, date range, customer)
```

---

### `endpoint-builder`

Cria rotas de API usando `Route::controller()` com agrupamento. Detecta se o projeto usa JWT ou Sanctum e aplica o middleware de autenticacao correto. Garante imports com `use` no topo do arquivo.

**Exemplos:**

```text
@whyll-agents:endpoint-builder Create CRUD routes for ProductController

@whyll-agents:endpoint-builder Add authenticated routes for OrderController with custom endpoint GET /orders/pending

@whyll-agents:endpoint-builder Create public routes (no auth) for CategoryController
```

---

### `test-builder`

Cria testes PHPUnit de feature para endpoints de API usando `DatabaseTransactions`. Suporta autenticacao via JWT (Bearer token) ou Sanctum (`actingAs`). Inclui testes para CRUD, validacao e autenticacao.

**Exemplos:**

```text
@whyll-agents:test-builder Create feature tests for ProductController API endpoints

@whyll-agents:test-builder Create tests for OrderController including validation and authentication checks

@whyll-agents:test-builder Create JWT-based tests for ArticleController with custom filter tests
```

---

### `job-builder`

Cria Jobs com `ShouldQueue`, logica de retry, timeout, backoff e tags para Horizon. Inclui metodos `handle()`, `failed()` e `tags()`.

**Exemplos:**

```text
@whyll-agents:job-builder Create a ProcessOrderJob that calculates totals and sends notification

@whyll-agents:job-builder Create a GenerateReportJob with 5 retries and 300s timeout

@whyll-agents:job-builder Create a SyncInventoryJob dispatched on the "processing" queue with 2min delay
```

---

### `resource-builder`

Cria JsonResource e ResourceCollection com campos explicitos, datas em ISO 8601, relacionamentos condicionais via `whenLoaded()` e contagens via `whenCounted()`. Inclui anotacao `@mixin` para IDE.

**Exemplos:**

```text
@whyll-agents:resource-builder Create Resource and Collection for Product with category relationship

@whyll-agents:resource-builder Create OrderResource with items count and user relationship

@whyll-agents:resource-builder Create ArticleResource with computed field "is_published" and tags relationship
```

---

### `seeder-builder`

Cria Factory com estados (states) e Seeder usando Faker. Reutiliza relacionamentos existentes com `recycle()`.

**Exemplos:**

```text
@whyll-agents:seeder-builder Create Factory and Seeder for Product with active/draft states

@whyll-agents:seeder-builder Create Factory for Order with states: pending, processing, completed

@whyll-agents:seeder-builder Create Factory and Seeder for Article recycling existing Categories and Users
```

---

### `event-builder`

Cria Events, Listeners e configuracao de Broadcasting com Reverb. Suporta eventos padrao e broadcastable com canais privados. Listeners sao enfileirados com retry.

**Exemplos:**

```text
@whyll-agents:event-builder Create OrderPlaced event with a SendOrderConfirmation listener

@whyll-agents:event-builder Create a broadcastable ProductUpdated event on a private channel

@whyll-agents:event-builder Create InvoicePaid event with two listeners: SendReceipt and UpdateAccountBalance
```

---

### `jwt-auth-builder`

Configura autenticacao JWT completa com `tymon/jwt-auth`: login, logout, refresh, me, forgot/reset password, update profile e audits do usuario. Implementa single-session via token cache. Detecta ACL e inclui roles/permissions quando presente.

**Exemplos:**

```text
@whyll-agents:jwt-auth-builder Setup complete JWT authentication for the project

@whyll-agents:jwt-auth-builder Configure JWT auth with password reset flow and profile update

@whyll-agents:jwt-auth-builder Setup JWT authentication with ACL role/permissions support in me() endpoint
```

---

### `acl-builder`

Configura sistema ACL modular de 3 niveis: User -> Role -> Module -> Permission. Cria models, migrations, traits, middleware, seeders e configuracao do Spatie Permission com UUID.

**Exemplos:**

```text
@whyll-agents:acl-builder Setup complete modular ACL system for the project

@whyll-agents:acl-builder Add new modules "invoice" and "report" to existing ACL with all permissions

@whyll-agents:acl-builder Create a "Manager" role with limited permissions on user and order modules
```

---

### `audit-builder`

Configura auditoria com `owen-it/laravel-auditing` e suporte a UUID. Cria resolvers customizados (IP, UserAgent, URL), resources de audit e verifica se os models estao corretamente configurados.

**Exemplos:**

```text
@whyll-agents:audit-builder Setup complete auditing system with UUID support

@whyll-agents:audit-builder Check audit configuration in all models and report issues

@whyll-agents:audit-builder Add Auditable trait and configuration to the Invoice model
```

---

### `horizon-builder`

Configura Laravel Horizon com Redis PhpRedis: instalacao, `config/horizon.php`, supervisors, balanceamento, deploy e configuracao de filas Redis.

**Exemplos:**

```text
@whyll-agents:horizon-builder Setup Horizon with auto-balancing for production and simple mode for local

@whyll-agents:horizon-builder Configure Horizon with queues: default, notifications, processing with different timeouts

@whyll-agents:horizon-builder Setup Horizon deployment with Supervisor config and metrics schedule
```

---

## Fixers (auditar e corrigir)

### `quality-checker`

Audita o codigo Laravel e gera prompts formatados para os agentes fixers. Verifica Controllers, Services, Repositories, Models, FormRequests e Resources contra os padroes do projeto. Detecta ACL e valida middleware.

**Exemplos:**

```text
@whyll-agents:quality-checker Audit the entire Product feature (Controller, Service, Repository, Model, Resource)

@whyll-agents:quality-checker Scan all Controllers for logic violations (try-catch, Log::, direct queries)

@whyll-agents:quality-checker Audit Order feature and generate fix prompts for each component
```

---

### `controller-fixer`

Remove toda logica do Controller, delegando para o Service. Elimina try-catch, Log::, queries diretas e `response()->json()`. Adiciona middleware ACL quando o projeto possui sistema de permissoes.

**Exemplos:**

```text
@whyll-agents:controller-fixer Fix ProductController - remove try-catch blocks and direct queries

@whyll-agents:controller-fixer Fix OrderController - add ACL middleware and delegate to OrderService

@whyll-agents:controller-fixer Fix ArticleController - replace Request with ArticleRequest and return JsonResource
```

---

### `service-fixer`

Garante que o Service estende `Service`, usa Repository para queries, retorna Resources e tem tratamento de erro com `Log::channel('services')`.

**Exemplos:**

```text
@whyll-agents:service-fixer Fix ProductService - use ProductRepository instead of direct DB queries

@whyll-agents:service-fixer Fix OrderService - return OrderResource instead of Model instances

@whyll-agents:service-fixer Fix InvoiceService - add ErrorResource handling and proper logging
```

---

### `repository-fixer`

Converte uso de `DB::table()`, `DB::select()` e `DB::raw()` para `Model::query()` com Eloquent, `whereHas()` e `with()`.

**Exemplos:**

```text
@whyll-agents:repository-fixer Fix ProductRepository - convert DB::table to Product::query()

@whyll-agents:repository-fixer Fix OrderRepository - replace JOIN with whereHas relationship

@whyll-agents:repository-fixer Fix ReportRepository - convert DB::raw() queries to Eloquent scopes
```

---

### `model-fixer`

Adiciona traits obrigatorias, configuracao UUID, `uniqueIds()`, corrige FKs de `_id` para `_uuid` e adiciona chaves explicitas nos relacionamentos.

**Exemplos:**

```text
@whyll-agents:model-fixer Fix Product model - add missing HasUuids and AutoIncrementId traits

@whyll-agents:model-fixer Fix Order model - change category_id to category_uuid and fix relationship keys

@whyll-agents:model-fixer Fix Invoice model - add Auditable trait and uniqueIds() method
```

---

### `formrequest-fixer`

Corrige FormRequests para tratar tanto criacao quanto atualizacao. Adiciona `authorize()`, converte pipes para array syntax e usa `Rule::unique()->ignore($uuid, 'uuid')` em updates.

**Exemplos:**

```text
@whyll-agents:formrequest-fixer Fix ProductRequest - add PUT/PATCH handling with UUID ignore

@whyll-agents:formrequest-fixer Fix OrderRequest - convert pipe validation to array syntax

@whyll-agents:formrequest-fixer Fix ArticleRequest - add authorize() and unique rule handling for updates
```

---

### `resource-fixer`

Remove `parent::toArray()`, adiciona campos explicitos, datas em ISO com null-safe operator, `whenLoaded()` para relacionamentos e `whenCounted()` para contagens. Adiciona `@mixin`.

**Exemplos:**

```text
@whyll-agents:resource-fixer Fix ProductResource - replace parent::toArray with explicit fields

@whyll-agents:resource-fixer Fix OrderResource - add toISOString() for dates and whenLoaded for relations

@whyll-agents:resource-fixer Fix ArticleResource - add @mixin annotation and whenCounted for comments
```
