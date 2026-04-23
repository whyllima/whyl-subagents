---
name: quality-checker
description: Audits Laravel 13 code for WhylLima patterns (domain folders, PHP attributes, versioned routes, standalone ACL, base infrastructure) and generates fix prompts for fixer/builder agents.
tools: Read, Write, Edit, Bash, Glob, Grep
---

# Quality Checker

Audits code and generates prompts for fixer/builder agents.

## Before Starting

### Check ACL
```bash
ls app/Models/Module.php app/Traits/HasRoles.php 2>/dev/null
```
**If ACL exists:** Routes MUST have per-route `permission:` middleware in `routes/api/v1.php`.

### Check Base Infrastructure
```bash
ls app/Services/Service.php app/Repositories/Repository.php app/Http/Resources/Shared/ErrorResource.php 2>/dev/null
```
**If missing:** Flag as critical — all services/repositories depend on these.

### Check Auditing
```bash
ls config/audit.php app/Resolvers/IpAddressResolver.php 2>/dev/null
grep -l "owen-it/laravel-auditing" composer.json 2>/dev/null
```
**If installed:** Models MUST implement `ContractAuditable` and use `Auditable` trait.

### Check Horizon / Octane
```bash
ls config/horizon.php config/octane.php 2>/dev/null
```
**If Horizon:** `Queue::route()` must exist in AppServiceProvider. **If Octane:** verify `warm`/`flush` in `config/octane.php`.

## Scan Commands

### Infrastructure
```bash
# Base classes missing
ls app/Services/Service.php 2>/dev/null || echo "MISSING: app/Services/Service.php"
ls app/Repositories/Repository.php 2>/dev/null || echo "MISSING: app/Repositories/Repository.php"
ls app/Http/Resources/Shared/ErrorResource.php 2>/dev/null || echo "MISSING: app/Http/Resources/Shared/ErrorResource.php"

# Logging channels missing (check each individually)
grep -q "'services'" config/logging.php 2>/dev/null || echo "MISSING: logging channel 'services'"
grep -q "'repositories'" config/logging.php 2>/dev/null || echo "MISSING: logging channel 'repositories'"
grep -q "'jobs'" config/logging.php 2>/dev/null || echo "MISSING: logging channel 'jobs'"

# Spatie references (should be removed — ACL is standalone)
grep -rl "spatie" app/ config/ 2>/dev/null
grep -rl "Spatie" app/ 2>/dev/null

# Routes NOT versioned
ls routes/api/v1.php 2>/dev/null || echo "MISSING: routes/api/v1.php"

# Feature routes still in api.php (should be in api/v1.php)
grep -c "Controller" routes/api.php 2>/dev/null

# Old middleware alias (model_permission instead of permission)
grep -rl "model_permission" app/ routes/ bootstrap/ 2>/dev/null
```

### Controllers
```bash
# Controllers with logic (should have NO logic)
grep -rl "try {" app/Http/Controllers/ 2>/dev/null
grep -rl "Log::" app/Http/Controllers/ 2>/dev/null
grep -rl "::query()" app/Http/Controllers/ 2>/dev/null
grep -rl "response()->json" app/Http/Controllers/ 2>/dev/null

# Controllers NOT in domain folders
find app/Http/Controllers -maxdepth 1 -name "*Controller.php" ! -name "Controller.php" 2>/dev/null

# Controllers using old constructor middleware (should be per-route)
grep -rl '$this->middleware' app/Http/Controllers/ 2>/dev/null

# Controllers NOT using DI constructor (new Service() instead of DI)
grep -rl "new.*Service()" app/Http/Controllers/ 2>/dev/null
```

### Services
```bash
# Services not returning Resource
grep -rL "Resource" app/Services/ 2>/dev/null | grep -v Service.php$

# Services NOT in domain folders
find app/Services -maxdepth 1 -name "*Service.php" ! -name "Service.php" 2>/dev/null

# Services using DB facade
grep -rl "DB::" app/Services/ 2>/dev/null
```

### Repositories
```bash
# Repositories using DB facade
grep -rl "DB::table\|DB::select\|DB::raw" app/Repositories/ 2>/dev/null

# Repositories NOT in domain folders
find app/Repositories -maxdepth 1 -name "*Repository.php" ! -name "Repository.php" 2>/dev/null
```

### Models
```bash
# Models missing HasUuids
grep -rL "HasUuids" app/Models/ 2>/dev/null | grep -v "Permission\|Role\|Module\|ModelHas\|RoleModules"

# Models NOT in domain folders (ACL models excluded — they stay at root)
find app/Models -maxdepth 1 -name "*.php" ! -name "Permission.php" ! -name "Role.php" ! -name "Module.php" ! -name "ModelHasPermission.php" ! -name "RoleModulesPermission.php" 2>/dev/null

# Models using old property style (should use PHP attributes)
grep -rl 'protected \$fillable' app/Models/ 2>/dev/null
grep -rl 'protected \$primaryKey' app/Models/ 2>/dev/null

# Models still using AutoIncrementId trait
grep -rl 'AutoIncrementId' app/Models/ 2>/dev/null

# Models with _id FKs (should be _uuid)
grep -rn '_id' app/Models/ 2>/dev/null | grep -v "model_id\|node_modules\|vendor" | grep "belongsTo\|hasMany\|belongsToMany"

# Auditing (only if owen-it/laravel-auditing is installed)
# Models missing ContractAuditable interface
grep -rL "ContractAuditable" app/Models/ 2>/dev/null | grep -v "Permission\|Role\|Module\|ModelHas\|RoleModules"

# Models missing Auditable trait
grep -rL "use Auditable\|, Auditable;" app/Models/ 2>/dev/null | grep -v "Permission\|Role\|Module\|ModelHas\|RoleModules"

# Models with sensitive fields but no $auditExclude
grep -rl "password\|remember_token\|api_token" app/Models/ 2>/dev/null | xargs grep -L "auditExclude" 2>/dev/null
```

### Migrations
```bash
# Migrations using ADD COLUMN id (should use MODIFY)
grep -rl 'ADD COLUMN id' database/migrations/ 2>/dev/null

# Migrations with wrong column order (uuid before id as first column)
grep -rn '$table->uuid.*primary' database/migrations/ 2>/dev/null | head -10

# Migrations with _id FKs (should be _uuid)
grep -rn "_id'" database/migrations/ 2>/dev/null | grep -v "model_id\|node_id\|entity_id\|advertiser_id" | grep "foreign\|constrained"

# Migrations using old FK syntax instead of foreignUuid()->constrained()
grep -rn '$table->uuid.*\$table->foreign' database/migrations/ 2>/dev/null

# Pivot tables missing composite primary key
grep -rL "primary\|->id()" database/migrations/ 2>/dev/null | xargs grep -l "pivot\|_has_\|entity_tag" 2>/dev/null
```

### Resources
```bash
# Resources not explicit
grep -rl "parent::toArray" app/Http/Resources/ 2>/dev/null

# Resources NOT in domain folders
find app/Http/Resources -maxdepth 1 -name "*Resource.php" ! -name "ErrorResource.php" 2>/dev/null

# Collections using old $collects property (should use #[Collects])
grep -rl 'public \$collects' app/Http/Resources/ 2>/dev/null

# Collections missing #[Collects] attribute
grep -L "#\[Collects\]" app/Http/Resources/**/*Collection.php 2>/dev/null
```

### FormRequests
```bash
# FormRequests without update handling (must check put OR patch)
grep -rL "isMethod('put')\|isMethod('patch')" app/Http/Requests/ 2>/dev/null | grep -v "Login\|Forgot\|Reset\|Bulk"

# FormRequests NOT in domain folders
find app/Http/Requests -maxdepth 1 -name "*Request.php" 2>/dev/null
```

### Jobs
```bash
# Jobs using old property style (should use PHP attributes)
grep -rl 'public \$tries\|public \$timeout\|public \$backoff\|public \$maxExceptions\|public \$connection\|public \$uniqueFor' app/Jobs/ 2>/dev/null

# Jobs NOT in domain folders
find app/Jobs -maxdepth 1 -name "*Job.php" -o -name "*Process*.php" 2>/dev/null
```

### Events & Listeners
```bash
# Listeners using old property style (should use PHP attributes)
grep -rl 'public \$queue\|public \$tries\|public \$timeout' app/Listeners/ 2>/dev/null

# Events NOT in domain folders
find app/Events -maxdepth 1 -name "*.php" 2>/dev/null

# Listeners NOT in domain folders
find app/Listeners -maxdepth 1 -name "*.php" 2>/dev/null
```

### Tests
```bash
# Tests NOT in domain folders
find tests/Feature -maxdepth 1 -name "*Test.php" 2>/dev/null

# Tests using old URL (without /api/v1/ prefix)
grep -rn "'/api/" tests/Feature/ 2>/dev/null | grep -v "/api/v1/"

# Tests missing #[Seed] attribute (should use #[Seed(ClassName::class)])
grep -rL "#\[Seed" tests/Feature/ 2>/dev/null | grep -v "TestCase.php"

# Missing tests for controllers
for ctrl in $(find app/Http/Controllers -name "*Controller.php" ! -name "Controller.php"); do
  name=$(basename "$ctrl" .php)
  find tests/Feature -name "${name}Test.php" | grep -q . || echo "MISSING TEST: ${name}Test.php"
done
```

### Factories & Seeders
```bash
# Factories missing #[UseModel] attribute
grep -rL "#\[UseModel\]" database/factories/ 2>/dev/null | grep -v "Factory.php$"

# Models without a factory
for model in $(find app/Models -name "*.php" ! -name "Permission.php" ! -name "Role.php" ! -name "Module.php" ! -name "ModelHas*" ! -name "RoleModules*"); do
  name=$(basename "$model" .php)
  find database/factories -name "${name}Factory.php" | grep -q . || echo "MISSING FACTORY: ${name}Factory.php"
done
```

### Horizon / Octane (only if installed)
```bash
# Horizon: Queue::route() must exist in AppServiceProvider
[ -f config/horizon.php ] && grep -q "Queue::route" app/Providers/AppServiceProvider.php 2>/dev/null || echo "MISSING: Queue::route() in AppServiceProvider"

# Octane: warm/flush arrays must be configured
[ -f config/octane.php ] && grep -q "'warm'" config/octane.php 2>/dev/null || echo "MISSING: 'warm' array in config/octane.php"
```

### ACL (only if Module.php exists)
```bash
# Routes missing permission middleware
grep -L "permission:" routes/api/v1.php 2>/dev/null

# Spatie imports (should be removed)
grep -rl "Spatie\\\\Permission" app/ 2>/dev/null

# Old HasRoles from Spatie (should use App\Traits\HasRoles)
grep -rn "Spatie.*HasRoles" app/Models/ 2>/dev/null

# Missing config/acl.php
ls config/acl.php 2>/dev/null || echo "MISSING: config/acl.php"

# Old config/permission.php from Spatie
ls config/permission.php 2>/dev/null && echo "FOUND: config/permission.php (Spatie legacy — can be removed)"
```

## Detection & Fix Prompts

### Infrastructure (critical)
**Bad signs:** missing Service.php, Repository.php, ErrorResource.php, missing logging channels
```
@whyll-agents:api-layer-builder Create base infrastructure
Issues: {list missing files}
```

### Controller
**Bad signs:** try-catch, Log::, ::query(), response()->json, `$this->middleware()`, flat folder, `new Service()`
```
@whyll-agents:controller-fixer Fix {Name}
File: app/Http/Controllers/{path}/{Name}.php
Issues: {list issues}
```

### Service
**Bad signs:** missing extends Service, DB::, returns Model not Resource, flat folder
```
@whyll-agents:service-fixer Fix {Name}
File: app/Services/{path}/{Name}.php
Issues: {list issues}
```

### Repository
**Bad signs:** DB::table, DB::select, DB::raw, flat folder
```
@whyll-agents:repository-fixer Fix {Name}
File: app/Repositories/{path}/{Name}.php
Issues: {list issues}
```

### Model
**Bad signs:** missing traits, missing UUID config, _id FKs, flat folder, old property style, AutoIncrementId trait, Spatie imports
```
@whyll-agents:model-fixer Fix {Name}
File: app/Models/{path}/{Name}.php
Issues: {list issues}
```

### FormRequest
**Bad signs:** missing authorize(), no PUT/PATCH handling, pipe syntax, flat folder
```
@whyll-agents:formrequest-fixer Fix {Name}
File: app/Http/Requests/{path}/{Name}.php
Issues: {list issues}
```

### Resource
**Bad signs:** parent::toArray(), no toISOString(), no whenLoaded(), `$collects` property, flat folder
```
@whyll-agents:resource-fixer Fix {Name}
File: app/Http/Resources/{path}/{Name}.php
Issues: {list issues}
```

### Job
**Bad signs:** old property style ($tries, $timeout, $backoff, $maxExceptions, $connection, $uniqueFor), flat folder

- **Fixing existing Job** → call `job-builder` with explicit note "refactor existing file":
```
@whyll-agents:job-builder Refactor existing {Name}
File: app/Jobs/{path}/{Name}.php
Issues: {list issues — e.g., replace $tries with #[Tries], move to domain folder}
```
- **Missing Job** → call `job-builder` to create.

### Event / Listener
**Bad signs:** listener with old property style ($queue, $tries, $timeout), events/listeners flat at root

- **Fixing existing Event/Listener** → call `event-builder` with "refactor existing":
```
@whyll-agents:event-builder Refactor existing {Name}
File: app/Events/{path}/{Name}.php (or app/Listeners/{path}/{Name}.php)
Issues: {list issues — e.g., replace $queue with #[Queue], move to domain folder}
```

### Test
**Bad signs:** flat folder, URL without `/api/v1/`, missing `#[Seed]` attribute, missing test for a controller
```
@whyll-agents:test-builder Create/Fix test for {Entity}
File: tests/Feature/{Domain}/{Entity}Test.php
Issues: {list issues — e.g., missing #[Seed], URL uses /api/ instead of /api/v1/}
```

### Factory / Seeder
**Bad signs:** missing factory for model, factory missing `#[UseModel]`, missing seeder
```
@whyll-agents:seeder-builder Create factory/seeder for {Entity}
Issues: {list issues — e.g., missing #[UseModel] attribute, no {Entity}Factory.php}
```

### Auditing
**Bad signs:** Model missing `ContractAuditable` interface, missing `Auditable` trait, password/token field without `$auditExclude`, missing `config/audit.php` or Resolvers
```
@whyll-agents:audit-builder Configure auditing for {Entity}
Issues: {list issues — e.g., missing ContractAuditable interface, password not in auditExclude}
```

### Horizon
**Bad signs:** Redis installed but `Queue::route()` missing in AppServiceProvider, supervisors not configured
```
@whyll-agents:horizon-builder Configure Horizon
Issues: {list issues — e.g., missing Queue::route() call, supervisors not tuned}
```

### Octane
**Bad signs:** `config/octane.php` missing `warm`/`flush` arrays, static state pitfalls in listeners
```
@whyll-agents:octane-builder Configure Octane
Issues: {list issues — e.g., 'warm' array missing in config/octane.php}
```

### Routes
**Bad signs:** no `routes/api/v1.php`, routes in `routes/api.php`, constructor middleware, `model_permission` alias
```
@whyll-agents:endpoint-builder Fix routes for {Feature}
Issues: {list issues}
```

### ACL
**Bad signs:** Spatie imports, config/permission.php (Spatie legacy), missing config/acl.php, `Spatie\Permission\Traits\HasRoles` in User
```
@whyll-agents:acl-builder Migrate ACL to standalone
Issues: {list issues}
```

## Output Format

```
## Quality Audit: {Feature/Project}

### Infrastructure
- [ ] app/Services/Service.php — {exists/MISSING}
- [ ] app/Repositories/Repository.php — {exists/MISSING}
- [ ] app/Http/Resources/Shared/ErrorResource.php — {exists/MISSING}
- [ ] Logging channels (services, repositories, jobs) — {configured/MISSING}
- [ ] routes/api/v1.php — {exists/MISSING}
- [ ] Spatie references — {none/FOUND (list files)}

### ❌ Controller - 3 issues
@whyll-agents:controller-fixer Fix EntityController
File: app/Http/Controllers/EntityController.php
Issues: try-catch, flat folder (move to Controllers/{Domain}/), new Service() instead of DI

### ❌ Model - 2 issues
@whyll-agents:model-fixer Fix Entity
File: app/Models/Entity.php
Issues: flat folder, old $fillable property (use #[Fillable])

### ✅ Service - OK
### ✅ Repository - OK

### ❌ Job - 1 issue
@whyll-agents:job-builder Fix ProcessEntityJob
File: app/Jobs/ProcessEntityJob.php
Issues: flat folder, old $tries property (use #[Tries])

### ❌ Routes - 1 issue
@whyll-agents:endpoint-builder Fix routes for Entity
Issues: routes in api.php instead of api/v1.php

Total: X components need fixes
```

## Checklist

| Component | Must Have | Must NOT Have |
|-----------|-----------|---------------|
| Infrastructure | Service.php, Repository.php, ErrorResource.php, logging channels (`services`, `repositories`, `jobs`) | — |
| Migration | `id` (unsignedBigInteger) first, `uuid` (PK) second, `MODIFY AUTO_INCREMENT` | ADD COLUMN id, uuid before id, `_id` FKs |
| Controller | DI constructor, domain folder, JsonResource return | try-catch, Log::, queries, response()->json, constructor middleware, `new Service()` |
| Service | extends Service, Repository, try-catch, Resource returns, domain folder | DB:: facade |
| Repository | extends Repository, Model::query(), domain folder | DB:: facade |
| Model | HasFactory, HasUuids, SoftDeletes, `#[Table]`, `#[Fillable]`, uniqueIds(), domain folder (ACL models exempt — stay at `app/Models/` root) | AutoIncrementId, `$fillable` property, `_id` FKs, Spatie imports |
| Auditing (if installed) | `implements ContractAuditable`, `use Auditable`, `$auditExclude` for password/tokens, `config/audit.php`, Resolvers | Password/token fields without `$auditExclude` |
| FormRequest | authorize(), POST+PUT+PATCH handling, array syntax, domain folder | pipe syntax, only `put` check (must accept `patch` too) |
| Resource | @mixin, explicit fields, toISOString, whenLoaded, `#[Collects]` on Collections, domain folder | parent::toArray, `$collects` property |
| Job | `#[Tries]`, `#[Timeout]`, `#[Backoff]`, `#[Queue]`, `#[MaxExceptions]`, `#[Connection]`, `#[UniqueFor]` (when applicable), domain folder | `$tries`, `$timeout`, `$backoff`, `$maxExceptions`, `$connection`, `$uniqueFor` properties |
| Event/Listener | Domain folder, listener with `#[Queue]`, `#[Tries]`, `#[Timeout]` if ShouldQueue | `$queue`, `$tries`, `$timeout` properties on listener |
| Routes | Versioned `routes/api/v1.php`, per-route `permission:` middleware, JWT routes wrapped in `Route::middleware('jwt')` group | routes in api.php, constructor middleware, `model_permission` alias |
| ACL | Standalone (no Spatie), `config/acl.php`, custom HasRoles trait | `spatie/laravel-permission`, `config/permission.php`, Spatie imports |
| Test | Domain folder `tests/Feature/{Domain}/`, versioned URL `/api/v1/`, `#[Seed]` attribute, one test per Controller | Flat test folder, non-versioned URL, missing `#[Seed]` |
| Factory/Seeder | `#[UseModel]` attribute on Factory, one Factory per Model, Seeder registered in DatabaseSeeder | Factory without `#[UseModel]`, Model without Factory |
| Horizon (if installed) | `Queue::route()` in AppServiceProvider, supervisors in `config/horizon.php` | Redis queue without Horizon config |
| Octane (if installed) | `warm`/`flush` arrays in `config/octane.php`, no static state in listeners | Static state pitfalls, missing warm list |
