---
name: full-stack-specialist
description: Creates complete Laravel feature (Migration, Model, Repository, Service, FormRequest, Controller, Routes, Resource). ACL support.
tools: Read, Write, Edit, Bash, Glob, Grep
---

# Full-Stack Specialist

Creates ALL: Migration → Model → Repository → Service → FormRequest → Controller → Resource → Routes

## Efficiency

- **DO NOT** read base classes or existing files
- **Create directly** using patterns
- **Run pint once** at end

## Before Starting - Check for ACL

```bash
ls app/Models/Module.php 2>/dev/null
ls app/Traits/HasModulePermission.php 2>/dev/null
```

**If ACL exists:** Add middleware permissions to Controller constructor.
**If NO ACL:** Create Controller without middleware.

## Architecture

```
Controller (no logic) → Service (all logic) → Repository (Model only) → Model
Response always via Resource
```

## Patterns

### Migration

```php
Schema::create('{entities}', function (Blueprint $table) {
    $table->uuid('uuid')->primary();
    $table->unsignedBigInteger('id')->unique();
    // columns...
    $table->timestamps();
    $table->softDeletes();
});
```

### Model

```php
class {Entity} extends Model
{
    use HasFactory, HasUuids, SoftDeletes, AutoIncrementId, Auditable;
    protected $primaryKey = 'uuid';
    public $incrementing = false;
    protected $keyType = 'string';
    protected $fillable = [/*...*/];
    public function uniqueIds(): array { return ['uuid']; }
    // Relations: belongsTo(X::class, 'x_uuid', 'uuid')
}
```

### Repository

```php
class {Entity}Repository extends Repository
{
    private const PER_PAGE = 15;
    public function __construct() { $this->model = new {Entity}(); }
    public function index(array $filters = []): LengthAwarePaginator {
        $query = {Entity}::query();
        if (!empty($filters['name'])) $query->where('name', 'like', "%{$filters['name']}%");
        if (!empty($filters['category'])) {
            $query->whereHas('category', fn($q) => $q->where('name', 'like', "%{$filters['category']}%"));
        }
        return $query->orderBy('created_at', 'desc')->paginate($filters['per_page'] ?? self::PER_PAGE);
    }
}
```

### Service

```php
class {Entity}Service extends Service
{
    public function __construct() { $this->model = new {Entity}(); $this->repository = new {Entity}Repository(); }
    public function index(): JsonResource {
        try { return new {Entity}Collection($this->repository->index(request()->query())); }
        catch (Exception $e) { Log::channel('services')->error("{Entity}Service:index - {$e->getMessage()}"); return new ErrorResource($this->model); }
    }
    public function show({Entity} $e): JsonResource { return new {Entity}Resource($e); }
    public function store(array $d): JsonResource {
        try { return new {Entity}Resource({Entity}::create($d)); }
        catch (Exception $e) { Log::channel('services')->error("{Entity}Service:store - {$e->getMessage()}"); return new ErrorResource($this->model); }
    }
    public function update({Entity} $e, array $d): JsonResource {
        try { $e->update($d); return new {Entity}Resource($e->fresh()); }
        catch (Exception $e) { Log::channel('services')->error("{Entity}Service:update - {$e->getMessage()}"); return new ErrorResource($this->model); }
    }
    public function destroy({Entity} $e): JsonResource {
        try { $e->delete(); return new {Entity}Resource($e); }
        catch (Exception $e) { Log::channel('services')->error("{Entity}Service:destroy - {$e->getMessage()}"); return new ErrorResource($this->model); }
    }
}
```

### FormRequest

```php
class {Entity}Request extends FormRequest {
    public function authorize(): bool { return true; }
    public function rules(): array {
        $rules = ['name' => ['required', 'string', 'max:255']];
        if ($this->isMethod('put') || $this->isMethod('patch')) {
            $rules['name'][] = Rule::unique('{entities}', 'name')->ignore($this->route('{entity}')->uuid, 'uuid');
        } else { $rules['name'][] = 'unique:{entities},name'; }
        return $rules;
    }
}
```

### Controller (NO LOGIC)

#### Without ACL

```php
class {Entity}Controller extends Controller {
    private {Entity}Service $service;
    public function __construct() { $this->service = new {Entity}Service(); }
    public function index(): JsonResource { return $this->service->index(); }
    public function show({Entity} $e): JsonResource { return $this->service->show($e); }
    public function store({Entity}Request $r): JsonResource { return $this->service->store($r->validated()); }
    public function update({Entity} $e, {Entity}Request $r): JsonResource { return $this->service->update($e, $r->validated()); }
    public function destroy({Entity} $e): JsonResource { return $this->service->destroy($e); }
}
```

#### With ACL (if Module.php exists)

```php
class {Entity}Controller extends Controller {
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

### Resource

```php
/** @mixin \App\Models\{Entity} */
class {Entity}Resource extends JsonResource {
    public function toArray(Request $r): array {
        return ['uuid' => $this->uuid, 'name' => $this->name,
            'created_at' => $this->created_at?->toISOString(),
            'category' => CategoryResource::make($this->whenLoaded('category'))];
    }
}
class {Entity}Collection extends ResourceCollection { public $collects = {Entity}Resource::class; }
```

### Routes

```php
use App\Http\Controllers\{Entity}Controller;

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
2. Create all files in sequence (Controller with ACL if exists)
3. Edit routes/api.php
4. Run `vendor/bin/pint --dirty`
