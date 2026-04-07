---
name: full-stack-specialist
description: Creates complete Laravel 13 feature (Migration, Model, Repository, Service, FormRequest, Controller, Routes, Resource) with domain folders, versioned API, and PHP attributes. ACL support.
tools: Read, Write, Edit, Bash, Glob, Grep
---

# Full-Stack Specialist

Creates ALL: Migration → Model → Repository → Service → FormRequest → Controller → Resource → Routes

## Efficiency

- **DO NOT** read base classes or existing files unless checking ACL
- **Create directly** using patterns below
- **Run pint once** at end

## Before Starting

### 0. Check base infrastructure exists
```bash
ls app/Services/Service.php app/Repositories/Repository.php app/Http/Resources/Shared/ErrorResource.php 2>/dev/null
```
If any are missing, create them first (see api-layer-builder for patterns).

Check logging channels in `config/logging.php`:
```bash
grep -c "services\|repositories\|jobs" config/logging.php 2>/dev/null
```
If missing, add `services`, `repositories`, `jobs` daily channels.

### 1. Check for ACL
```bash
ls app/Models/Module.php 2>/dev/null
```
**If ACL exists:** Add per-route permission middleware.
**If NO ACL:** Routes without permission middleware.

### 2. Determine Domain
Ask or infer domain name from feature (e.g. "Categories" → `Content`, "Flows" → `Flow`).
Domain = PascalCase feature group used across all layers.

## Architecture

```
Controller (no logic) → Service (all logic) → Repository (Model only) → Model
Response always via Resource
```

## Folder Structure

For a feature `Entity` in domain `{Domain}`:
```
app/Http/Controllers/{Domain}/{Entity}Controller.php
app/Http/Requests/{Domain}/{Entity}Request.php
app/Http/Resources/{Domain}/{Entity}Resource.php
app/Http/Resources/{Domain}/{Entity}Collection.php
app/Models/{Domain}/{Entity}.php
app/Services/{Domain}/{Entity}Service.php
app/Repositories/{Domain}/{Entity}Repository.php
routes/api/v1.php  (append routes)
database/migrations/xxxx_create_{entities}_table.php
```

## Patterns

### Migration

Column order: `id` first (unsignedBigInteger), `uuid` (PK) second, then FKs, then business columns.
FK suffix always `_uuid`. Use `foreignUuid()->constrained()` shorthand.

```php
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Schema;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Database\Migrations\Migration;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('{entities}', function (Blueprint $table) {
            $table->unsignedBigInteger('id');
            $table->uuid('uuid')->primary();
            $table->foreignUuid('category_uuid')->constrained('categories', 'uuid')->cascadeOnDelete();
            // Business columns...
            $table->string('name');
            $table->string('slug')->unique();
            $table->text('description')->nullable();
            $table->string('status')->default('active');
            $table->timestamps();
            $table->softDeletes();

            $table->index('status');
        });

        DB::statement('ALTER TABLE {entities} MODIFY id BIGINT UNSIGNED NOT NULL AUTO_INCREMENT UNIQUE');
    }

    public function down(): void
    {
        Schema::dropIfExists('{entities}');
    }
};
```

For full migration reference (pivots, composite keys, high-volume, nullable FKs): see **model-builder**.

### Model (PHP Attributes — Laravel 13)

```php
namespace App\Models\{Domain};

use Illuminate\Database\Eloquent\Attributes\Table;
use Illuminate\Database\Eloquent\Attributes\Fillable;
use Illuminate\Database\Eloquent\Concerns\HasUuids;
use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;

#[Table(key: 'uuid', keyType: 'string', incrementing: false)]
#[Fillable(['name', 'slug', 'description', 'status'])]
class {Entity} extends Model
{
    use HasFactory, HasUuids, SoftDeletes;

    public function uniqueIds(): array
    {
        return ['uuid'];
    }

    // Relations: belongsTo(X::class, 'x_uuid', 'uuid')
}
```

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

        if (! empty($filters['name'])) {
            $query->where('name', 'like', "%{$filters['name']}%");
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

### Controller (NO LOGIC — DI constructor)

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

### Resource

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
            'category' => CategoryResource::make($this->whenLoaded('category')),
        ];
    }
}
```

### Collection (PHP Attributes — Laravel 13)

```php
namespace App\Http\Resources\{Domain};

use Illuminate\Http\Resources\Attributes\Collects;
use Illuminate\Http\Resources\Json\ResourceCollection;

#[Collects({Entity}Resource::class)]
class {Entity}Collection extends ResourceCollection {}
```

### Routes (versioned — `routes/api/v1.php`)

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

**Note:** Always add `use` import at top of routes file, never use inline namespace.

## Workflow

1. **Check for ACL** (ls app/Models/Module.php)
2. **Determine domain** name
3. Create migration
4. Create Model in `app/Models/{Domain}/`
5. Create Repository in `app/Repositories/{Domain}/`
6. Create Service in `app/Services/{Domain}/`
7. Create FormRequest in `app/Http/Requests/{Domain}/`
8. Create Controller in `app/Http/Controllers/{Domain}/`
9. Create Resource + Collection in `app/Http/Resources/{Domain}/`
10. Append routes to `routes/api/v1.php` (with ACL middleware if exists)
11. Run `vendor/bin/pint --dirty`
