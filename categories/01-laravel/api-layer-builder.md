---
name: api-layer-builder
description: Creates Laravel 13 API layer (Repository, Service, FormRequest, Controller, Routes, Resource) for existing models with domain folders, versioned routes, and PHP attributes. ACL support.
tools: Read, Write, Edit, Bash, Glob, Grep
---

# API Layer Builder

Creates API layer for an EXISTING model. Model and Migration must already exist.

## Efficiency

- **DO NOT** read base classes
- **Read the Model** to understand fields and relationships
- **Create directly** using patterns

## Before Starting

### 1. Check for ACL
```bash
ls app/Models/Module.php 2>/dev/null
```
**If ACL exists:** Add per-route permission middleware in routes.
**If NO ACL:** Routes without permission middleware.

### 2. Find the Model
```bash
find app/Models -name "{Entity}.php" 2>/dev/null
```
Read it to get: namespace (domain), fillable fields, relationships.

### 3. Determine Domain
Extract from model namespace: `App\Models\{Domain}\{Entity}` → domain is `{Domain}`.

### 4. Check base classes exist
```bash
ls app/Services/Service.php app/Repositories/Repository.php app/Http/Resources/Shared/ErrorResource.php 2>/dev/null
```
If missing, create them (see Base Classes section below).

## Architecture

```
Controller (no logic) → Service (all logic) → Repository (Model only) → Model
Response always via Resource
```

## Creates (in order)

1. Repository → `app/Repositories/{Domain}/{Entity}Repository.php`
2. Service → `app/Services/{Domain}/{Entity}Service.php`
3. FormRequest → `app/Http/Requests/{Domain}/{Entity}Request.php`
4. Controller → `app/Http/Controllers/{Domain}/{Entity}Controller.php`
5. Resource → `app/Http/Resources/{Domain}/{Entity}Resource.php`
6. Collection → `app/Http/Resources/{Domain}/{Entity}Collection.php`
7. Routes → append to `routes/api/v1.php`

## Patterns

### Repository

```php
namespace App\Repositories\{Domain};

use App\Models\{Domain}\{Entity};
use App\Repositories\Repository;
use Illuminate\Pagination\LengthAwarePaginator;

class {Entity}Repository extends Repository
{
    private const PER_PAGE = 15;

    public function __construct()
    {
        $this->model = new {Entity}();
    }

    public function index(array $filters = []): LengthAwarePaginator
    {
        $query = {Entity}::query();

        if (! empty($filters['search'])) {
            $query->where('name', 'like', "%{$filters['search']}%");
        }

        return $query->orderBy('created_at', 'desc')
            ->paginate($filters['per_page'] ?? self::PER_PAGE);
    }
}
```

### Service

```php
namespace App\Services\{Domain};

use App\Models\{Domain}\{Entity};
use App\Repositories\{Domain}\{Entity}Repository;
use App\Http\Resources\{Domain}\{Entity}Collection;
use App\Http\Resources\{Domain}\{Entity}Resource;
use App\Http\Resources\Shared\ErrorResource;
use App\Services\Service;
use Exception;
use Illuminate\Http\Resources\Json\JsonResource;
use Illuminate\Support\Facades\Log;

class {Entity}Service extends Service
{
    public function __construct()
    {
        $this->model = new {Entity}();
        $this->repository = new {Entity}Repository();
    }

    public function index(): JsonResource
    {
        try {
            return new {Entity}Collection($this->repository->index(request()->query()));
        } catch (Exception $e) {
            Log::channel('services')->error("{Entity}Service:index - {$e->getMessage()}");
            return new ErrorResource($this->model);
        }
    }

    public function show({Entity} $entity): JsonResource
    {
        return new {Entity}Resource($entity);
    }

    public function store(array $data): JsonResource
    {
        try {
            return new {Entity}Resource({Entity}::create($data));
        } catch (Exception $e) {
            Log::channel('services')->error("{Entity}Service:store - {$e->getMessage()}");
            return new ErrorResource($this->model);
        }
    }

    public function update({Entity} $entity, array $data): JsonResource
    {
        try {
            $entity->update($data);
            return new {Entity}Resource($entity->fresh());
        } catch (Exception $e) {
            Log::channel('services')->error("{Entity}Service:update - {$e->getMessage()}");
            return new ErrorResource($this->model);
        }
    }

    public function destroy({Entity} $entity): JsonResource
    {
        try {
            $entity->delete();
            return new {Entity}Resource($entity);
        } catch (Exception $e) {
            Log::channel('services')->error("{Entity}Service:destroy - {$e->getMessage()}");
            return new ErrorResource($this->model);
        }
    }
}
```

### FormRequest

```php
namespace App\Http\Requests\{Domain};

use Illuminate\Foundation\Http\FormRequest;
use Illuminate\Validation\Rule;

class {Entity}Request extends FormRequest
{
    public function authorize(): bool
    {
        return true;
    }

    public function rules(): array
    {
        $rules = [
            'name' => ['required', 'string', 'max:255'],
        ];

        if ($this->isMethod('put') || $this->isMethod('patch')) {
            $rules['name'][] = Rule::unique('{entities}', 'name')
                ->ignore($this->route('{entity}')->uuid, 'uuid');
        } else {
            $rules['name'][] = 'unique:{entities},name';
        }

        return $rules;
    }
}
```

### Controller (DI, no logic)

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

### Resource + Collection

```php
namespace App\Http\Resources\{Domain};

use Illuminate\Http\Request;
use Illuminate\Http\Resources\Json\JsonResource;

/** @mixin \App\Models\{Domain}\{Entity} */
class {Entity}Resource extends JsonResource
{
    public function toArray(Request $request): array
    {
        return [
            'uuid' => $this->uuid,
            'name' => $this->name,
            'created_at' => $this->created_at?->toISOString(),
            'updated_at' => $this->updated_at?->toISOString(),
        ];
    }
}
```

```php
namespace App\Http\Resources\{Domain};

use Illuminate\Http\Resources\Attributes\Collects;
use Illuminate\Http\Resources\Json\ResourceCollection;

#[Collects({Entity}Resource::class)]
class {Entity}Collection extends ResourceCollection {}
```

### Routes (`routes/api/v1.php`)

#### With ACL
```php
use App\Http\Controllers\{Domain}\{Entity}Controller;

Route::controller({Entity}Controller::class)->prefix('{entities}')->group(function () {
    Route::get('/', 'index')->middleware('permission:index-{entity}');
    Route::get('/{entity}', 'show')->middleware('permission:show-{entity}');
    Route::post('/', 'store')->middleware('permission:store-{entity}');
    Route::put('/{entity}', 'update')->middleware('permission:update-{entity}');
    Route::delete('/{entity}', 'destroy')->middleware('permission:delete-{entity}');
});
```

#### Without ACL
```php
use App\Http\Controllers\{Domain}\{Entity}Controller;

Route::controller({Entity}Controller::class)->prefix('{entities}')->group(function () {
    Route::get('/', 'index');
    Route::get('/{entity}', 'show');
    Route::post('/', 'store');
    Route::put('/{entity}', 'update');
    Route::delete('/{entity}', 'destroy');
});
```

## Base Classes (create if missing)

### `app/Services/Service.php`

```php
namespace App\Services;

use Illuminate\Database\Eloquent\Model;

abstract class Service
{
    protected Model $model;
    protected $repository;
}
```

### `app/Repositories/Repository.php`

```php
namespace App\Repositories;

use Illuminate\Database\Eloquent\Model;

abstract class Repository
{
    protected Model $model;
}
```

### `app/Http/Resources/Shared/ErrorResource.php`

```php
namespace App\Http\Resources\Shared;

use Illuminate\Http\Request;
use Illuminate\Http\Resources\Json\JsonResource;

class ErrorResource extends JsonResource
{
    public function __construct(
        $resource,
        private string $message = 'An error occurred.',
        private int $statusCode = 500,
    ) {
        parent::__construct($resource);
    }

    public function toArray(Request $request): array
    {
        return [
            'error' => true,
            'message' => $this->message,
        ];
    }

    public function withResponse(Request $request, $response): void
    {
        $response->setStatusCode($this->statusCode);
    }
}
```

## Logging Channels (add to `config/logging.php` if missing)

```php
'channels' => [
    // ... existing channels
    'services' => [
        'driver' => 'daily',
        'path' => storage_path('logs/services.log'),
        'level' => 'debug',
        'days' => 14,
    ],
    'repositories' => [
        'driver' => 'daily',
        'path' => storage_path('logs/repositories.log'),
        'level' => 'debug',
        'days' => 14,
    ],
    'jobs' => [
        'driver' => 'daily',
        'path' => storage_path('logs/jobs.log'),
        'level' => 'debug',
        'days' => 14,
    ],
],
```

## Workflow

1. Check for ACL
2. Read existing Model (get domain, fields, relationships)
3. Check base classes exist (create if missing)
4. Check logging channels exist (add if missing)
5. Create Repository, Service, FormRequest, Controller, Resource, Collection
6. Append routes to `routes/api/v1.php`
7. Run `vendor/bin/pint --dirty`
