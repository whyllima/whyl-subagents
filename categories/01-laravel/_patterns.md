# WhylLima Patterns Reference

Shared patterns for all agents. DO NOT read this file - patterns are embedded in each agent.

## Stack

- **PHP:** 8.3+
- **Laravel:** 13
- **Auth:** JWT (tymon/jwt-auth) or Sanctum v4
- **Queue:** Horizon v5
- **Real-time:** Reverb v1
- **Testing:** PHPUnit ^12 / Pest ^4
- **Auditing:** owen-it/laravel-auditing
- **AI:** laravel/ai (AI SDK)

## Architecture

```
Controller (no logic) → Service (all logic) → Repository (Model queries) → Model
                                                                         ↓
                                    Response via Resource ←───────────────
```

## Domain-Based Folder Structure

All layers organized by domain (feature area):

```
app/
├── Http/
│   ├── Controllers/{Domain}/     # e.g. Controllers/Content/RuleController.php
│   ├── Requests/{Domain}/        # e.g. Requests/Content/StoreRuleRequest.php
│   └── Resources/{Domain}/       # e.g. Resources/Content/RuleResource.php
├── Models/{Domain}/              # e.g. Models/Content/Rule.php
├── Services/{Domain}/            # e.g. Services/Content/RuleService.php
├── Repositories/{Domain}/        # e.g. Repositories/Content/RuleRepository.php
├── Jobs/{Domain}/                # e.g. Jobs/Content/ProcessRuleJob.php
└── Events/{Domain}/              # e.g. Events/Content/RuleCreated.php
```

Domain name = PascalCase feature group (e.g. `Content`, `Auth`, `Billing`, `Learning`).

## Versioned API Routes

```
routes/
├── api.php           # Health check only, no versioning
└── api/
    └── v1.php        # All v1 endpoints
```

Permission middleware applied per-route (not in constructor):
```php
Route::get('/', 'index')->middleware('permission:index-{entity}');
```

## Rules Summary

| Component | Rule |
|-----------|------|
| Controller | NO logic, DI constructor, delegates to Service, returns JsonResource |
| Service | ALL logic, uses Repository, returns Resource/ErrorResource |
| Repository | Model::query() only, NEVER DB::, throws on error |
| Migration | id (unsignedBigInteger) first, uuid (PK) second, MODIFY AUTO_INCREMENT; FK: `foreignUuid('x_uuid')->constrained('x', 'uuid')` |
| Model | HasFactory, HasUuids, SoftDeletes; UUID PK; _uuid FKs; PHP attributes preferred |
| FormRequest | Handles POST + PUT/PATCH, Rule::unique()->ignore() |
| Resource | Explicit fields, toISOString(), whenLoaded(), #[Collects] on Collection |
| Routes | Versioned in routes/api/v1.php, per-route permission middleware |
