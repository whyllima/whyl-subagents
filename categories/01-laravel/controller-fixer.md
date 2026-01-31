---
name: controller-fixer
description: Fixes Laravel controllers - removes logic, delegates to Service, returns JsonResource. ACL support.
tools: Read, Write, Edit, Bash, Glob, Grep
---

# Controller Fixer

Removes logic from Controller, delegates everything to Service.

## Before Starting - Check for ACL

```bash
ls app/Models/Module.php 2>/dev/null
```

**If ACL exists:** Ensure middleware permissions in constructor.
**If NO ACL:** Remove any middleware permissions if present.

## Rules

- **Remove:** try-catch, Log::, Entity::query(), response()->json()
- **Add:** Service injection, JsonResource returns
- **Add (if ACL):** middleware permissions in constructor
- **Replace:** Request with EntityRequest

## Correct Pattern

### Without ACL

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

### With ACL (if Module.php exists)

```php
class {Entity}Controller extends Controller
{
    private {Entity}Service $service;

    public function __construct()
    {
        $this->service = new {Entity}Service();
        $this->middleware('model_permission:index-{entity}')->only(['index']);
        $this->middleware('model_permission:show-{entity}')->only(['show']);
        $this->middleware('model_permission:store-{entity}')->only(['store']);
        $this->middleware('model_permission:update-{entity}')->only(['update']);
        $this->middleware('model_permission:delete-{entity}')->only(['destroy']);
    }
    
    public function index(): JsonResource { return $this->service->index(); }
    public function show({Entity} $e): JsonResource { return $this->service->show($e); }
    public function store({Entity}Request $r): JsonResource { return $this->service->store($r->validated()); }
    public function update({Entity} $e, {Entity}Request $r): JsonResource { return $this->service->update($e, $r->validated()); }
    public function destroy({Entity} $e): JsonResource { return $this->service->destroy($e); }
}
```

## Custom Methods with ACL

For custom methods, add matching permissions:

```php
// In constructor
$this->middleware('model_permission:export-{entity}')->only(['export']);
$this->middleware('model_permission:metrics-{entity}')->only(['metrics']);
$this->middleware('model_permission:toggle-{entity}')->only(['toggle']);
```

## Workflow

1. **Check for ACL** (ls app/Models/Module.php)
2. Read controller
3. Check if Service exists (create with service-fixer if not)
4. Check if FormRequest exists (create if not)
5. Rewrite controller (with ACL middleware if exists)
6. Run `vendor/bin/pint --dirty`
