---
name: quality-checker
description: Audits Laravel code for WhylLima patterns and generates fix prompts for fixer agents.
tools: Read, Write, Edit, Bash, Glob, Grep
---

# Quality Checker

Audits code and generates prompts for fixer agents.

## Scan Commands

```bash
grep -l "try {" app/Http/Controllers/           # Controllers with logic
grep -l "DB::table" app/Repositories/           # Repos using DB facade
grep -l "return \$" app/Services/ | xargs grep -L "Resource"  # Services not returning Resource
grep -L "AutoIncrementId" app/Models/*.php      # Models missing traits
grep -l "parent::toArray" app/Http/Resources/   # Resources not explicit
grep -L "isMethod('put')" app/Http/Requests/    # FormRequests without update handling
```

## Detection & Fix Prompts

### Controller
**Bad signs:** try-catch, Log::, Entity::where, response()->json
```
@whyll-agents:controller-fixer Fix {Name}
File: app/Http/Controllers/{Name}.php
Issues: {list issues}
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
| Service | extends Service, Repository, try-catch, Resource returns | DB:: facade |
| Repository | extends Repository, Model::query(), whereHas | DB:: facade |
| Model | 5 traits, UUID PK config, uniqueIds(), _uuid FKs | _id FKs |
| FormRequest | authorize(), POST+PUT handling, array syntax | pipe syntax |
| Resource | @mixin, explicit fields, toISOString, whenLoaded | parent::toArray, id field |
