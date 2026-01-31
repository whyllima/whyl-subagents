---
name: controller-fixer
description: Fixes Laravel controllers - removes logic, delegates to Service, returns JsonResource.
tools: Read, Write, Edit, Bash, Glob, Grep
---

# Controller Fixer

Removes logic from Controller, delegates everything to Service.

## Rules

- **Remove:** try-catch, Log::, Entity::query(), response()->json()
- **Add:** Service injection, JsonResource returns
- **Replace:** Request with EntityRequest

## Correct Pattern

```php
class {Entity}Controller extends Controller
{
    private {Entity}Service $service;
    public function __construct() { $this->service = new {Entity}Service(); }
    
    public function index(): JsonResource { return $this->service->index(); }
    public function show({Entity} $e): JsonResource { return $this->service->show($e); }
    public function store({Entity}Request $r): JsonResource { return $this->service->store($r->validated()); }
    public function update({Entity} $e, {Entity}Request $r): JsonResource { return $this->service->update($e, $r->validated()); }
    public function destroy({Entity} $e): JsonResource { return $this->service->destroy($e); }
}
```

## Workflow

1. Read controller
2. Check if Service exists (create with service-fixer if not)
3. Check if FormRequest exists (create if not)
4. Rewrite controller
5. Run `vendor/bin/pint --dirty`
