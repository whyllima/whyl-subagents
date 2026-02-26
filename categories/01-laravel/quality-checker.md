---
name: quality-checker
description: Audits Laravel code for WhylLima patterns and generates fix prompts for fixer agents. ACL support.
tools: Read, Write, Edit, Bash, Glob, Grep
---

# Quality Checker

Audits code and generates prompts for fixer agents.

## Before Starting - Check for ACL

```bash
ls app/Models/Module.php 2>/dev/null
```

**If ACL exists:** Controllers MUST have `model_permission` middleware.
**If NO ACL:** Controllers should NOT have `model_permission` middleware.

## Scan Commands

```bash
grep -l "try {" app/Http/Controllers/           # Controllers with logic
grep -l "DB::table" app/Repositories/           # Repos using DB facade
grep -l "return \$" app/Services/ | xargs grep -L "Resource"  # Services not returning Resource
grep -L "HasUuids" app/Models/*.php              # Models missing UUID traits
grep -l "parent::toArray" app/Http/Resources/   # Resources not explicit
grep -L "isMethod('put')" app/Http/Requests/    # FormRequests without update handling

# ACL checks (only if Module.php exists)
grep -L "model_permission" app/Http/Controllers/*.php  # Controllers missing ACL
```

## Detection & Fix Prompts

### Controller
**Bad signs:** try-catch, Log::, Entity::where, response()->json
**If ACL exists:** missing `model_permission` middleware
```
@whyll-agents:controller-fixer Fix {Name}
File: app/Http/Controllers/{Name}.php
Issues: {list issues}
ACL: {yes/no - based on Module.php existence}
```

### Service
**Bad signs:** missing extends Service, DB::, returns Model not Resource
```
@whyll-agents:service-fixer Fix {Name}
File: app/Services/{Name}.php
Issues: {list issues}
```

### Repository
**Bad signs:** DB::table, DB::select, DB::raw
```
@whyll-agents:repository-fixer Fix {Name}
File: app/Repositories/{Name}.php
Issues: {list issues}
```

### Model
**Bad signs:** missing traits, missing UUID config, _id instead of _uuid
```
@whyll-agents:model-fixer Fix {Name}
File: app/Models/{Name}.php
Issues: {list issues}
```

### FormRequest
**Bad signs:** missing authorize(), no PUT/PATCH handling, pipe syntax
```
@whyll-agents:formrequest-fixer Fix {Name}
File: app/Http/Requests/{Name}.php
Issues: {list issues}
```

### Resource
**Bad signs:** parent::toArray(), no toISOString(), no whenLoaded()
```
@whyll-agents:resource-fixer Fix {Name}
File: app/Http/Resources/{Name}.php
Issues: {list issues}
```

## Output Format

```
## Quality Audit: {Feature}

### ❌ Controller - 3 issues
@whyll-agents:controller-fixer Fix EntityController
File: app/Http/Controllers/EntityController.php
Issues: try-catch, Log::, direct queries

### ✅ Service - OK
### ✅ Repository - OK
### ❌ Model - 2 issues
...

Total: X components need fixes
```

## Checklist

| Component | Must Have | Must NOT Have |
|-----------|-----------|---------------|
| Controller | Service injection, JsonResource return | try-catch, Log::, queries, response()->json |
| Controller (ACL) | model_permission middleware for each action | - |
| Service | extends Service, Repository, try-catch, Resource returns | DB:: facade |
| Repository | extends Repository, Model::query(), whereHas | DB:: facade |
| Model | HasFactory, HasUuids, SoftDeletes, UUID PK config, uniqueIds(), _uuid FKs | AutoIncrementId trait, _id FKs |
| FormRequest | authorize(), POST+PUT handling, array syntax | pipe syntax |
| Resource | @mixin, explicit fields, toISOString, whenLoaded | parent::toArray, id field |

## ACL Middleware Format

```php
$this->middleware('model_permission:{action}-{entity}')->only(['{method}']);
```

| Action | Method |
|--------|--------|
| index | index |
| show | show |
| store | store |
| update | update |
| delete | destroy |
| export | export |
| metrics | metrics |
| toggle | toggle |
