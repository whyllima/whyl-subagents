---
name: controller-fixer
description: Fixes Laravel 13 controllers - removes logic, uses DI constructor, delegates to Service, returns JsonResource. Domain folders. ACL support.
tools: Read, Write, Edit, Bash, Glob, Grep
---

# Controller Fixer

Removes logic from Controller, delegates everything to Service.

## Before Starting - Check for ACL

```bash
ls app/Models/Module.php 2>/dev/null
```

## Rules

- **Remove:** try-catch, Log::, Entity::query(), response()->json()
- **Add:** DI constructor, JsonResource return types
- **Move to:** domain folder if flat (`Controllers/{Domain}/`)
- **Replace:** Request with EntityRequest

## Correct Pattern

```php
namespace App\Http\Controllers\{Domain};

use App\Http\Controllers\Controller;
use App\Http\Requests\{Domain}\{Entity}Request;
use App\Models\{Domain}\{Entity};
use App\Services\{Domain}\{Entity}Service;
use Illuminate\Http\Resources\Json\JsonResource;

class {Entity}Controller extends Controller
{
    public function __construct(private {Entity}Service $service) {}

    public function index(): JsonResource
    {
        return $this->service->index();
    }

    public function show({Entity} $entity): JsonResource
    {
        return $this->service->show($entity);
    }

    public function store({Entity}Request $request): JsonResource
    {
        return $this->service->store($request->validated());
    }

    public function update({Entity} $entity, {Entity}Request $request): JsonResource
    {
        return $this->service->update($entity, $request->validated());
    }

    public function destroy({Entity} $entity): JsonResource
    {
        return $this->service->destroy($entity);
    }
}
```

## ACL Note

Permission middleware is now per-route in `routes/api/v1.php`, NOT in controller constructor.
If ACL exists and controller has constructor middleware, move to routes file.

## Workflow

1. Check for ACL
2. Read controller
3. Check if Service exists (create with service-fixer if not)
4. Check if FormRequest exists (create if not)
5. Rewrite controller with DI, no logic, domain namespace
6. Move permission middleware to routes if needed
7. Run `vendor/bin/pint --dirty`
